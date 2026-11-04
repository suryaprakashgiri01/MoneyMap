# Stage 03 — User Financial Profile & Products

**Branch:** `stage/03-user-financial-profile`
**Spec:** FR-PROF-01..05, FR-METR-01..04, FR-EMRG-01..04, FR-GOAL-01..04, FR-CAT-01, FR-RATE-01..04,
FR-HOLD-01..04, FR-CMP-01..04, FR-SCN-01..04, FR-TRAN-01..04, ADR-0008, ADR-0010

## Objective

Everything a user can record, and every deterministic answer computed from it — exposed via REST.

## Tasks

| ID | Task | Notes |
| --- | --- | --- |
| T03-01 | Profile tables + migration | profile, income history, expenses |
| T03-02 | Profile CRUD endpoints | validation, optimistic concurrency |
| T03-03 | Metrics endpoint | uses engine, persists `computed_result` |
| T03-04 | Emergency fund endpoint | essential-expense classification required |
| T03-05 | Goals CRUD + progress | required contribution, infeasible flag |
| T03-06 | Rate dataset + seed loader | YAML → `product_rates`, `verified_by` required |
| T03-07 | Rate resolution service | verified + fresh, `missing_input` / `stale` semantics |
| T03-08 | Holdings CRUD | type-specific validation, soft delete, audit |
| T03-09 | LIC policy details | 1:1 with holding, bonus assumption handling |
| T03-10 | Comparison endpoint | normalisation, `not_comparable`, no ranking |
| T03-11 | Scenario CRUD + diff | immutable snapshots, deltas |
| T03-12 | `computed_results` persistence | TTL 24 h, `result_ref` returned |
| T03-13 | Transparency endpoints | `/v1/meta/assumptions`, formula docs |
| T03-14 | Isolation tests per resource | GET/PATCH/DELETE cross-org → 404 |
| T03-15 | Contract tests | OpenAPI-driven |
| T03-16 | Audit events on mutations | no amounts in metadata |

## Exit criteria

- AC-METR, AC-EMRG, AC-GOAL, AC-RATE, AC-CMP, AC-SCN pass.
- No endpoint returns a number without provenance and `trace_id`.
- A missing rate produces 422 `missing_input`, never a default.
- Scenario creation provably does not mutate holdings.

## Out of scope

AI explanations, memory, UI.

## Risks

| Risk | Mitigation |
| --- | --- |
| Rate seed accuracy | `verified_by` + source URL + review checklist |
| Sparse `not_comparable` cases | explicit reason enum + tests |
| Endpoint sprawl | keep DTOs flat; one resource per router |
