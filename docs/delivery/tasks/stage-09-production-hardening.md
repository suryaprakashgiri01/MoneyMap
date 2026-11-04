# Stage 09 — Production Hardening

**Branch:** `stage/09-production-hardening`
**Spec:** release DoD (`docs/quality/definition-of-done.md` §3), SPEC-SEC-001 §9, SPEC-DLV-001 §11

## Objective

Make the MVP operable and defensible: security pass, performance pass, runbooks, and a rehearsed
rollback and restore.

## Tasks

| ID | Task | Notes |
| --- | --- | --- |
| T09-01 | Security review vs threat model | every threat mapped to a verified control |
| T09-02 | Dependency + container scan | `pip-audit`, `npm audit`, image scan, no unresolved criticals |
| T09-03 | Rate-limit + budget tuning | from real usage data |
| T09-04 | DB index + slow-query review | `EXPLAIN` on hot queries |
| T09-05 | k6 smoke load test | 50 VU, record p95 |
| T09-06 | Backup/restore drill | documented + executed |
| T09-07 | Runbooks | deploy, rollback, incident, rate refresh, key rotation |
| T09-08 | Error-budget review | alerts verified against real incidents |
| T09-09 | Data retention + purge job verification | computed results, deleted accounts |
| T09-10 | Full regression + eval run | stored report, thresholds met |
| T09-11 | Privacy notice + disclaimer legal review | pre-launch blocker |
| T09-12 | README finalisation | quickstart, limitations, architecture map |
| T09-13 | Release tag `v0.1.0` | annotated, immutable |

## Exit criteria

- Release DoD fully checked with evidence.
- Rollback executed once successfully in staging.
- Restore from snapshot executed once.
- Eval thresholds E-01..E-12 met; report stored.
- No known critical/high vulnerability outstanding.

## Out of scope

Multi-region, autoscaling, paid support tiers, SOC2/HIPAA-style certifications.

## Risks

| Risk | Mitigation |
| --- | --- |
| Hardening expands into feature work | task list is fixed; new ideas go to a post-launch backlog |
| Manual rate refresh forgotten | documented monthly task + staleness flags degrade safely |
| Single-instance availability | documented; Redis + second instance as first post-launch upgrade |
