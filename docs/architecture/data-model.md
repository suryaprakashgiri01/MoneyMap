# Data Model Proposal

**Document ID:** SPEC-ARCH-002 · **Stage:** 00 · **Status:** Approved v1.0
**Related:** ADR-0003 (Decimal), ADR-0010 (tenancy), ADR-0007 (memory)

## 1. Conventions

- Primary keys: `UUID v4` (`gen_random_uuid()`), never sequential integers exposed to clients.
- Timestamps: `TIMESTAMPTZ`, stored UTC, `created_at`/`updated_at` on every table.
- Money: `NUMERIC(18,2)`; interest rates: `NUMERIC(9,6)` (e.g. `0.071000`); ratios as integer basis
  points `NUMERIC`-free (`INTEGER`).
- Soft delete: `deleted_at TIMESTAMPTZ NULL` on user-facing content tables; hard delete via purge job.
- Tenant column: `org_id UUID NOT NULL REFERENCES orgs(id)` on every tenant-scoped table, indexed
  together with the natural lookup key.
- Enumerations: PostgreSQL native enums are avoided (migration pain); use `VARCHAR` + `CHECK`
  constraints with values mirrored in Python `StrEnum`.
- All `CHECK` constraints mirror Pydantic validation so the DB is a second line of defence.

## 2. Identity & access

### `orgs`
| Column | Type | Notes |
| --- | --- | --- |
| id | UUID PK | |
| name | TEXT | `Personal workspace` for MVP-created orgs |
| created_at | TIMESTAMPTZ | |

### `users`
| Column | Type | Notes |
| --- | --- | --- |
| id | UUID PK | |
| org_id | UUID FK → orgs | one personal org per user at MVP |
| email | CITEXT UNIQUE | normalised lowercase |
| password_hash | TEXT | Argon2id |
| display_name | TEXT NULL | |
| email_verified_at | TIMESTAMPTZ NULL | |
| last_login_at | TIMESTAMPTZ NULL | |
| deleted_at | TIMESTAMPTZ NULL | soft delete → purge |
| created_at / updated_at | TIMESTAMPTZ | |

Index: `(org_id)`, unique `(lower(email))` where `deleted_at IS NULL`.

### `refresh_tokens`
| Column | Type | Notes |
| --- | --- | --- |
| id | UUID PK | |
| user_id | UUID FK | |
| family_id | UUID | rotation family |
| token_hash | TEXT UNIQUE | never store raw token |
| issued_at | TIMESTAMPTZ | |
| expires_at | TIMESTAMPTZ | |
| revoked_at | TIMESTAMPTZ NULL | set on rotation/reuse detection |
| replaced_by | UUID NULL | chain for reuse detection |
| user_agent_hash / ip_hash | TEXT NULL | privacy-safe abuse signals |

Index: `(user_id, family_id)`, `(expires_at)`.

## 3. Financial profile

### `financial_profiles` (1:1 user)
| Column | Type | Notes |
| --- | --- | --- |
| id | UUID PK | |
| org_id, user_id | UUID | unique `(user_id)` |
| currency | CHAR(3) | `INR` only (CHECK) |
| income_type | VARCHAR | `salaried` \| `variable` |
| monthly_net_income | NUMERIC(18,2) NULL | null when `variable` (derived from history) |
| emergency_months | SMALLINT | default 6, CHECK 1..24 |
| assumed_return_rate | NUMERIC(9,6) NULL | user override for goal projections |
| created_at / updated_at | TIMESTAMPTZ | optimistic concurrency via `updated_at` |

### `income_history`
| Column | Type | Notes |
| --- | --- | --- |
| id | UUID PK | |
| org_id, user_id | UUID | |
| month | DATE | first day of month, unique `(user_id, month)` |
| net_amount | NUMERIC(18,2) | CHECK > 0 |

### `expense_items`
| Column | Type | Notes |
| --- | --- | --- |
| id | UUID PK | |
| org_id, user_id | UUID | |
| label | TEXT | ≤ 80 chars |
| category | VARCHAR | `housing`, `food`, `transport`, `education`, `health`, `insurance`, `debt`, `lifestyle`, `other` |
| amount | NUMERIC(18,2) | CHECK > 0 |
| periodicity | VARCHAR | `monthly` \| `annual` \| `one_off` |
| is_essential | BOOLEAN | default true |
| deleted_at | TIMESTAMPTZ NULL | |

Index: `(org_id, user_id, deleted_at)`.

## 4. Goals & emergency fund

### `goals`
| Column | Type | Notes |
| --- | --- | --- |
| id | UUID PK | |
| org_id, user_id | UUID | |
| name | TEXT | ≤ 80 |
| target_amount | NUMERIC(18,2) | CHECK > 0 |
| target_date | DATE | must be future on create |
| priority | SMALLINT | 1..5 |
| assumed_rate | NUMERIC(9,6) NULL | null → presented as 0% with `assumed` flag |
| status | VARCHAR | `active` \| `achieved` \| `archived` |
| deleted_at | TIMESTAMPTZ NULL | |

Emergency fund is **derived** (target = `emergency_months × essential_expenses`), not a stored goal.

## 5. Products, rates & holdings

### `product_rates` (append-only, versioned)
| Column | Type | Notes |
| --- | --- | --- |
| id | UUID PK | |
| product_type | VARCHAR | `fd`,`rd`,`ppf`,`post_office_td`,`post_office_rd`,`post_office_mis`,`post_office_scss`,`mutual_fund_sip`,`lic_policy` |
| variant | TEXT | e.g. `1y`, `general`, `senior_citizen` |
| rate_type | VARCHAR | `interest` \| `expected_return` \| `bonus` |
| value | NUMERIC(9,6) | annual, as decimal fraction |
| effective_from | DATE | |
| effective_to | DATE NULL | null = open-ended |
| source_name | TEXT | institution/scheme |
| source_url | TEXT | must be https |
| retrieved_at | TIMESTAMPTZ | |
| verified_by | TEXT | human reviewer id |
| is_assumption | BOOLEAN | true for illustrative expected returns (e.g. equity SIP) |
| created_at | TIMESTAMPTZ | |

Unique: `(product_type, variant, rate_type, effective_from)`. Append-only: updates create a new row
and close the previous `effective_to`.

### `holdings`
| Column | Type | Notes |
| --- | --- | --- |
| id | UUID PK | |
| org_id, user_id | UUID | |
| product_type | VARCHAR | CHECK in catalogue |
| label | TEXT NULL | user alias |
| principal | NUMERIC(18,2) NULL | lump sum / current invested |
| monthly_contribution | NUMERIC(18,2) NULL | SIP/RD |
| start_date | DATE | |
| tenure_months | SMALLINT NULL | |
| rate_id | UUID FK → product_rates NULL | sourced rate |
| user_rate | NUMERIC(9,6) NULL | user-supplied override (requires `rate_source='user'`) |
| rate_source | VARCHAR | `sourced` \| `user` |
| liquidity | VARCHAR | `liquid` \| `lock_in` \| `partial` |
| lock_in_until | DATE NULL | |
| maturity_date | DATE NULL | |
| maturity_value_entered | NUMERIC(18,2) NULL | for LIC/endowment |
| current_value | NUMERIC(18,2) NULL | |
| notes | TEXT NULL | untrusted text — never parsed as instructions |
| deleted_at | TIMESTAMPTZ NULL | |

CHECK: exactly one of (`rate_id`, `user_rate`) present. Index `(org_id, user_id, deleted_at)`.

### `lic_policy_details` (1:1 holding where `product_type='lic_policy'`)
| Column | Type | Notes |
| --- | --- | --- |
| id | UUID PK | |
| org_id, holding_id | UUID | unique `(holding_id)` |
| plan_name | TEXT NULL | free text, user-supplied |
| sum_assured | NUMERIC(18,2) | CHECK > 0 |
| annual_premium | NUMERIC(18,2) | CHECK > 0 |
| premium_term_years | SMALLINT | |
| policy_term_years | SMALLINT | |
| bonus_rate_id | UUID NULL | sourced bonus assumption |
| bonus_assumed | NUMERIC(9,6) NULL | if no source, flagged `assumed` |
| paid_premiums_to_date | NUMERIC(18,2) NULL | derived from start date if absent |
| surrender_value_entered | NUMERIC(18,2) NULL | informational only |

## 6. Scenarios & computed results

### `scenarios`
| Column | Type | Notes |
| --- | --- | --- |
| id | UUID PK | |
| org_id, user_id | UUID | |
| name | TEXT | ≤ 80 |
| horizon_months | SMALLINT | CHECK 1..600 |
| baseline_snapshot | JSONB | immutable copy of inputs used |
| status | VARCHAR | `draft` \| `saved` |
| created_at | TIMESTAMPTZ | |

### `scenario_changes`
| Column | Type | Notes |
| --- | --- | --- |
| id | UUID PK | |
| org_id, scenario_id | UUID FK | |
| source_holding_id | UUID NULL | null = from surplus |
| target_product_type | VARCHAR | |
| monthly_amount | NUMERIC(18,2) | CHECK > 0 |
| start_date | DATE | |
| duration_months | SMALLINT | |

### `computed_results` (server-side, feeds AI grounding)
| Column | Type | Notes |
| --- | --- | --- |
| id | UUID PK | this is the `result_ref` |
| org_id, user_id | UUID | |
| kind | VARCHAR | `metrics` \| `emergency_fund` \| `goal` \| `lic_analysis` \| `comparison` \| `scenario` |
| payload | JSONB | full result incl. provenance + assumptions |
| formula_ids | TEXT[] | |
| engine_version | TEXT | git sha or semver of domain layer |
| created_at | TIMESTAMPTZ | |
| expires_at | TIMESTAMPTZ | default +24h; purge job deletes |

Index `(org_id, user_id, created_at DESC)`, `(expires_at)`. The AI path may read only this table for
numbers (ADR-0004).

## 7. Memory

### `memory_entries`
| Column | Type | Notes |
| --- | --- | --- |
| id | UUID PK | |
| scope | VARCHAR | `user` \| `organization` \| `generic` |
| user_id | UUID NULL | required when scope=`user` |
| org_id | UUID NULL | required when scope∈(`user`,`organization`) |
| kind | VARCHAR | `preference` \| `goal_context` \| `planning_assumption` \| `terminology` \| `product_rule` |
| key | TEXT | short machine key, unique per (scope, owner, key) |
| content | TEXT | ≤ 2,000 chars, validated (no secrets/account numbers) |
| provenance | VARCHAR | `explicit_user_confirmation` \| `curated_seed` \| `admin_import` |
| source_ref | TEXT NULL | e.g. seed file path |
| is_active | BOOLEAN | default true |
| created_at / updated_at | TIMESTAMPTZ | |

CHECK: scope integrity (generic ⇒ both ids null). Indexes: `(user_id, kind, is_active)`,
`(org_id, kind, is_active)`.

## 8. Audit, feedback, events

### `audit_events`
`id`, `org_id`, `actor_user_id`, `action` (e.g. `holding.delete`), `entity_type`, `entity_id`,
`metadata JSONB` (no monetary values), `request_id`, `created_at`. Append-only.

### `security_events`
`id`, `event_type` (`login_failed`, `token_reuse`, `cross_org_attempt`, `rate_limited`),
`actor_hash`, `ip_hash`, `request_id`, `created_at`. No PII.

### `feedback_entries`
`id`, `org_id`, `user_id`, `trace_id`, `result_ref`, `rating` (`useful`|`not_useful`|`inaccurate`),
`comment TEXT NULL` (≤ 500, user-supplied, not parsed as instruction), `prompt_version`, `model`,
`created_at`.

Prompt templates are versioned **files** in `backend/app/ai/prompts/` (git + hash), not DB rows.

## 9. Relationship summary

```
orgs 1─n users 1─1 financial_profiles
users 1─n income_history, expense_items, goals, holdings, scenarios, memory_entries
holdings 1─1 lic_policy_details
product_rates 1─n holdings (rate_id)
scenarios 1─n scenario_changes
users 1─n computed_results ──(explain)── feedback_entries
```

## 10. Migration & seed policy

- Alembic, forward-only, one migration per stage PR, autogenerate reviewed by hand.
- Seed script loads `infra/seed/product_rates.yaml` (sourced, dated) and generic memory only.
- No user data in seeds. All fixtures synthetic (`infra/fixtures/`).
- Schema version asserted at startup (`/readyz` fails on mismatch).

## 11. Retention

| Data | Retention |
| --- | --- |
| `computed_results` | 24 h (configurable) |
| `refresh_tokens` | until expiry + 7 days |
| `audit_events`, `security_events` | 180 days |
| User content after deletion | hard-deleted ≤ 30 days |
| Langfuse traces | provider retention, redacted |

## 12. Open questions

1. Do we need `lic_policy_details.surrender_value_entered` in MVP? Retained but informational only.
2. Email verification is modelled but not required at MVP — confirm before Stage 01 auth work.
3. Shared orgs require a `memberships` table; deferred with ADR-0010.
