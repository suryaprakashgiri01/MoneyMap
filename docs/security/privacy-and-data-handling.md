# Privacy & Data Handling

**Document ID:** SPEC-SEC-002 · **Stage:** 00 · **Status:** Approved v1.0
**Related:** NFR-PRIV-01..07, ADR-0013, SPEC-SEC-001

## 1. Principle

Collect the minimum needed to compute a financial picture the user asked for; keep it only while it is
useful; never let it leave the system unredacted.

## 2. Data inventory

| Data | Purpose | Category | Stored in | Retention |
| --- | --- | --- | --- | --- |
| Email | Identity, login | PII | `users` | Life of account |
| Password hash (Argon2id) | Authentication | Secret | `users` | Life of account |
| Display name (optional) | UI greeting | PII | `users` | Life of account |
| Income and expense amounts | Core calculation | Financial | `financial_profiles`, `income_history`, `expense_items` | Life of account |
| Holdings, LIC policy details | Analysis | Financial | `holdings`, `lic_policy_details` | Life of account |
| Goals | Planning | Financial | `goals` | Life of account |
| Scenarios | What-if analysis | Financial | `scenarios`, `scenario_changes` | Life of account |
| Computed results | AI grounding source | Financial | `computed_results` | 24 h (configurable) |
| Memory entries | Personalisation | Financial/PII | `memory_entries` | Until user deletes |
| Feedback | Quality improvement | Behavioural | `feedback_entries` | 180 days |
| Audit events | Accountability | Behavioural | `audit_events` | 180 days |
| Security events | Abuse detection | Hashed identifiers | `security_events` | 180 days |
| Refresh tokens (hashed) | Sessions | Secret | `refresh_tokens` | Expiry + 7 d |
| Traces / logs | Debugging, reliability | Pseudonymous | OTel backend, Langfuse | Provider-defined (redacted) |

**Never collected in MVP:** bank credentials, account numbers, PAN/Aadhaar, card numbers, KYC
documents, uploaded statements, precise location, contacts, device fingerprints beyond IP/UA hashes.

## 3. Data flows leaving our trust domain

| Flow | What is sent | Safeguards |
| --- | --- | --- |
| LLM provider | Pruned result payload + ≤6 memory items + glossary terms | No email/name; no other users' data; no credentials; documented provider retention |
| Langfuse | Prompt version, model, tokens, latency, grounding outcome, pseudonymous trace id | No prompt text, no amounts |
| OTel backend | HTTP/DB span attributes from allow-list, pseudonymous user hash | Redactor, attribute allow-list, CI regex test |
| Hosting platform | Standard request metadata | TLS, no body logging |

No analytics SDK, ad pixel, or third-party script on authenticated pages (NFR-PRIV-07).

## 4. Minimisation rules in code

1. The context builder selects fields explicitly (no `SELECT *` into prompts).
2. Amounts are excluded from all log statements — a lint rule bans `logger.*(…amount…)` patterns in
   price-bearing modules, and review checks this.
3. Error messages reference field names, never values.
4. Prompt payloads are pruned per template (SPEC-ARCH-003 §5).
5. Feedback submissions never include computed amounts; they reference `trace_id`/`result_ref`.

## 5. User rights

| Right | Implementation |
| --- | --- |
| Access / portability | `GET /v1/me/export` returns the user's records as JSON (S) |
| Correction | Standard CRUD on all user records |
| Deletion | `DELETE /v1/me` soft-deletes immediately; purge job hard-deletes within 30 days |
| Memory control | List, deactivate, delete individual entries |
| Consent visibility | Memory confirmation UI shows exact stored text before saving |

Deletion covers: profile, income history, expenses, goals, holdings, LIC details, scenarios, computed
results, memory, feedback, refresh tokens. Audit and security events are retained with the user id
replaced by the pseudonymous hash.

## 6. Subprocessors

| Subprocessor | Role | Data |
| --- | --- | --- |
| LLM provider (configurable) | Explanation generation | Pruned financial payload |
| Langfuse | LLM observability | Metadata only |
| Telemetry backend | Traces/metrics | Pseudonymous metadata |
| Cloud host | Compute | Encrypted volume |
| Managed Postgres | Storage | All durable data (encrypted at rest) |
| Frontend host | Static/edge delivery | No financial data at edge |

Documented in the deployed app's privacy notice; changing a subprocessor is a release note item.

## 7. Legal / regulatory posture

- Positioned as an educational tool, not an adviser (see `product-scope.md`, `non-goals-and-disclaimers.md`).
- Consent-based processing: the user enters their own data to receive an analysis.
- No selling, sharing, or brokering of data; no affiliate links.
- Data residency: single region chosen at deploy time; documented in the privacy notice.
- This document is not legal advice; a jurisdiction-specific review is an open item before public launch.

## 8. Incident response (MVP scale)

1. Detect via error-rate/grounding-failure alarms or user report.
2. Contain: revoke tokens (`log out all` global switch), rotate secrets, disable AI path via config flag.
3. Assess scope using audit/security events (no amounts in logs, so scope is per-account).
4. Notify affected users if personal data was exposed.
5. Record a post-incident note with the fix and a regression test.

## 9. Open items

1. Formal privacy notice text and jurisdiction review (pre-launch blocker).
2. Whether to enable provider zero-retention mode by default (cost/quality trade-off to measure in Stage 04).
3. Export endpoint scope (S) — confirm whether it ships in MVP or Stage 09.
