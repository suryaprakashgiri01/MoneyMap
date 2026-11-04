# Stage 00 — Product Specification

**Branch:** `stage/00-product-spec` · **Status:** Complete (pending approval)
**Spec:** all documents under `docs/`

## Objective

Freeze an approved, testable specification baseline. No application code.

## Tasks

| ID | Task | Deliverable | Status |
| --- | --- | --- | --- |
| T00-01 | Repository inspection | environment note in stage report | done |
| T00-02 | Problem statement + scope + personas | `docs/product/*` | done |
| T00-03 | User stories + success metrics | `docs/product/*` | done |
| T00-04 | FR/NFR/AC catalogues | `docs/requirements/*` | done |
| T00-05 | System architecture + data model | `docs/architecture/*` | done |
| T00-06 | AI workflow + memory specs | `docs/architecture/*` | done |
| T00-07 | ADR-0001..0014 | `docs/architecture/decisions/*` | done |
| T00-08 | Threat model + privacy handling | `docs/security/*` | done |
| T00-09 | Test + evaluation strategy + DoD | `docs/quality/*` | done |
| T00-10 | Deployment strategy, roadmap, git workflow | `docs/delivery/*` | done |
| T00-11 | Stage task breakdown 00–09 | `docs/delivery/tasks/*` | done |
| T00-12 | README + repo rules | `README.md`, `AGENTS.md` | done |

## Exit criteria

- [x] Every user story maps to an FR; every FR maps to an AC.
- [x] ADRs cover all material decisions; no "TBD" in a normative statement.
- [x] Open questions collected and answered or explicitly deferred.
- [x] Repo initialised with correct git identity; branch/commit plan approved.

## Deferred to later stages

- Traceability matrix (generated in Stage 06).
- Privacy notice legal text (pre-launch, Stage 09).
- Email-verification requirement decision (Stage 01).

## Risks

| Risk | Mitigation |
| --- | --- |
| Spec becomes stale as code evolves | Stage DoD requires doc updates; ADRs for decisions |
| Over-specification slows delivery | Task files stay small; minor decisions documented inline |
| Python 3.15 local vs 3.12 target | `uv` pins 3.12/3.13; verify wheel availability in Stage 01 |
