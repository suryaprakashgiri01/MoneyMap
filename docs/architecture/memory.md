# Memory Specification

**Document ID:** SPEC-ARCH-004 · **Stage:** 00 · **Status:** Approved v1.0
**Related:** ADR-0007 (structured memory, no vectors), FR-MEM-01..05

## 1. What memory is (and is not)

Memory is a **small, explicit, structured store of facts the user chose to remember** or that the
product curates as shared knowledge. It is not a conversation log, not an embedding dump, and not an
implicit profile inferred from behaviour.

Hard rules:
1. Nothing is written without an explicit trigger (user confirmation UI, curated seed, admin import).
2. Nothing is written containing secrets, credentials, or account numbers.
3. Memory never becomes the source of a financial number — numbers come from the engine.
4. Retrieval is bounded, scoped, and logged.

## 2. Three scopes

### 2.1 User memory (`scope = user`)
- Private to one user; requires `user_id` + `org_id`.
- Examples: "prefers a 6-month emergency fund", "goal context: child's education in 2038",
  "wants comparisons expressed in monthly terms".
- Written only via `POST /v1/memory` from the confirmation UI.
- Deletable by the user, effective immediately.

### 2.2 Organization memory (`scope = organization`)
- Belongs to a workspace; requires `org_id`.
- MVP: the personal org. Contents are user-authored planning assumptions shared within that org
  (later: shared household/team workspaces).
- Never readable across `org_id` boundaries; cross-org read returns empty (not 403) and logs a
  security event.

### 2.3 Generic knowledge (`scope = generic`)
- Shared, non-personal: terminology, published product rules, educational definitions.
- `user_id` and `org_id` MUST be NULL (DB CHECK).
- Written only from curated seed files (`infra/seed/generic_memory.yaml`) with `source_ref`.
- Contains no private financial data by construction; validator rejects currency-amount patterns
  unless flagged `educational_example`.

## 3. Schema

See `data-model.md` §7. Fields that make the store auditable: `scope`, `kind`, `key`, `content`,
`provenance`, `source_ref`, `is_active`, `created_at`, `updated_at`.

`kind` values:
| kind | Used for | Scopes |
| --- | --- | --- |
| `preference` | presentation/planning preferences | user, organization |
| `goal_context` | durable user-stated context for a goal | user |
| `planning_assumption` | org-level assumed rates/conventions | organization |
| `terminology` | definitions ("XIRR") | generic |
| `product_rule` | published scheme rules (e.g. PPF 15-year term) | generic |

## 4. Write path

```
UI confirm → POST /v1/memory
  → auth + org
  → MemoryValidator:
      - scope/owner consistency
      - content length ≤ 2000
      - reject secrets: API keys, passwords, "account number", 9–18 digit runs, card patterns
      - reject instructions intended for the model (prompt-injection patterns)
      - reject imperative financial directives
  → upsert by (scope, owner, key)
  → audit_events row (action=memory.write, no content in metadata)
  → 201 with id
```

No other code path may insert into `memory_entries` except seeds and the admin import CLI.

## 5. Read path

```
ContextBuilder.build(result_payload, user, org)
  → user prefs: SELECT … WHERE user_id=:uid AND kind IN ('preference','goal_context')
                 AND is_active ORDER BY updated_at DESC LIMIT 3
  → org assumptions: WHERE org_id=:oid AND kind='planning_assumption' … LIMIT 3
  → generic terms: WHERE scope='generic' AND key = ANY(:needed_terms) LIMIT 5
  → assert every row's scope/owner matches the caller (defence in depth)
  → return typed MemoryBundle (not raw text)
```

Retrieval is **deterministic key lookup**, not semantic search. No embeddings, no pgvector, no
similarity threshold to tune. This is intentionally boring and testable.

## 6. Injection into prompts

- Memory items are rendered as a bulleted, clearly delimited data block:
  `<memory scope="user" kind="preference">…</memory>`, with a fixed instruction that these are
  data, not instructions.
- At most 6 items total per request; each truncated to 200 chars.
- Memory is **not** injected into refusal paths.

## 7. Update & deletion

| Action | Behaviour |
| --- | --- |
| Update | `PUT /v1/memory/{id}` — same validator; bumps `updated_at`; audit row |
| Deactivate | `is_active=false` — excluded from retrieval immediately |
| Delete | `DELETE /v1/memory/{id}` — hard delete, audit row without content |
| Account deletion | all user memory hard-deleted with the account |

Deletion is verified by a test asserting the subsequent context bundle no longer contains the key
(AC-MEM-02).

## 8. Consent & transparency

- The UI shows exactly what will be stored before storing it.
- A memory list view shows every entry with scope, creation time, and provenance.
- There is no "auto-learn" toggle in MVP. Auto-extraction is explicitly out of scope.

## 9. Why no vector store yet

A written requirement for semantic retrieval does not exist: users have < 20 memory rows, and the
prompt needs at most 3 user preferences keyed by kind. Adding pgvector would introduce embedding cost,
a provider dependency, a second consistency store, and an eval surface with no demonstrated benefit.

Re-open this decision only if: memory rows per user exceed ~100 **and** a measured case exists where
key-based retrieval returns the wrong items. That trigger requires an ADR amendment.

## 10. Self-learning boundary

- Memory updates change *personalisation*, not logic.
- Prompts, formulas, thresholds, and model config change only through human-reviewed PRs with eval runs.
- The product never claims the model trained itself (see `docs/product/non-goals-and-disclaimers.md`).

## 11. Test obligations

| Test | Assertion |
| --- | --- |
| T-MEM-01 | No row created without explicit confirm |
| T-MEM-02 | Secret/account-number patterns rejected |
| T-MEM-03 | Cross-user and cross-org reads return empty |
| T-MEM-04 | Deleted memory absent from next context bundle |
| T-MEM-05 | Injection-shaped memory text does not alter model instructions (eval) |
| T-MEM-06 | Retrieval caps enforced (≤3 + ≤3 + ≤5, ≤6 total injected) |
