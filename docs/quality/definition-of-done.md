# Definition of Done

**Document ID:** SPEC-QLT-003 · **Stage:** 00 · **Status:** Approved v1.0

Three levels: task, stage, release. An item is done only when every applicable box is checked **and**
verified, not assumed.

## 1. Task DoD

- [ ] Requirement IDs referenced (`FR-*`, `NFR-*`, `AC-*`, `ADR-*`) in the task file / PR body.
- [ ] Code follows layering (`api → services → domain`, no domain imports of infra/ai/api).
- [ ] Money handled as `Decimal`; no `float` in domain code.
- [ ] Inputs validated (Pydantic strict) and errors use stable machine codes.
- [ ] Org scoping applied to every query and covered by an isolation test.
- [ ] No secrets, real financial data, or PII in code, fixtures, seeds, logs, or commits.
- [ ] Tests added/updated: unit + integration as applicable; failing test written first where practical.
- [ ] `ruff`, `mypy`, and the test suite pass locally — **actually executed**.
- [ ] Docs updated if behaviour, API, config, or assumptions changed.
- [ ] Diff reviewed for unrelated changes; no drive-by refactors.
- [ ] Commit message conventional and scoped to the change.

## 2. Stage DoD

- [ ] All tasks in `docs/delivery/tasks/stage-NN-*.md` complete or explicitly deferred with reason.
- [ ] Stage acceptance criteria from `docs/requirements/acceptance-criteria.md` pass (executed).
- [ ] No skipped/xfail tests except documented, linked to an issue.
- [ ] Coverage gates met (`docs/quality/test-strategy.md` §9).
- [ ] Security checklist from `docs/security/threat-model.md` relevant to the stage verified.
- [ ] Observability: new endpoints/spans/logs have `request_id`; no amounts logged.
- [ ] Migrations reviewed, forward-only, and applied on a clean database in CI.
- [ ] `README.md` quickstart still accurate for a fresh clone.
- [ ] ADRs updated if a decision changed; no silent architectural drift.
- [ ] Stage report written: objective, spec refs, tasks, files, decisions, tests run + results,
      security/privacy notes, remaining work, suggested commits.
- [ ] Branch merged to `main` only after the gate; `PROGRESS.local.md` updated after the merge commit.

## 3. Release DoD (public MVP)

- [ ] All Stage 00–09 gates satisfied.
- [ ] Eval suite green at required thresholds (E-01..E-12) with a stored report.
- [ ] Full regression suite green on the release commit.
- [ ] PII/secret scans green; dependency audit has no unresolved criticals.
- [ ] Privacy notice and disclaimer text present and linked (DISC-01..DISC-05).
- [ ] Deployed URL reachable over HTTPS; health endpoints healthy; migrations applied.
- [ ] Rollback path documented and rehearsed once.
- [ ] Known limitations listed in the README.
- [ ] Backup/restore procedure documented; a restore has been tested.

## 4. Evidence rules

| Claim | Required evidence |
| --- | --- |
| "Tests pass" | Command output from this session / CI run link |
| "Coverage is X%" | Report artefact |
| "AI is grounded" | Eval report with `E-01 = 0` |
| "Isolation works" | Security test names + pass output |
| "Deployed" | URL + `/healthz` response |

Unverifiable claims are treated as not done. "It should work" is not a completion state.

## 5. Anti-patterns that block merge

- Adding a feature not covered by an FR.
- Hardcoding a rate, model name, or threshold in business logic.
- Falling back to a default when data is missing instead of returning `missing_input`.
- Logging request bodies or amounts.
- Silencing a failing test (`skip`, `xfail`, `# type: ignore`) without a linked issue.
- Widening an endpoint's accepted fields "for convenience".
- Committing generated artefacts, `.env` files, or dumps.
