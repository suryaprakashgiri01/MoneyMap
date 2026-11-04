# Stage 06 — Guardrails & Evaluation

**Branch:** `stage/06-guardrails-evals`
**Spec:** FR-GRD-01..05, FR-EVAL-01..03, FR-DISC-01, ADR-0011, SPEC-QLT-002

## Objective

Make the system's boundaries enforced and measurable: refusals, injection resistance, output policy,
and a repeatable eval harness.

## Tasks

| ID | Task | Notes |
| --- | --- | --- |
| T06-01 | Rule-based scope classifier | transactions, guarantees, timing, tax filing, "best product" |
| T06-02 | Static refusal table + endpoint behaviour | no model call on refusal |
| T06-03 | Forbidden-language scan | patterns from `non-goals-and-disclaimers.md` §4 |
| T06-04 | Disclaimer injection | DISC-01..05 server-side |
| T06-05 | Input fencing for user text | notes/memory as data blocks |
| T06-06 | PII output scan | email/phone/account-number patterns |
| T06-07 | AI budget + rate limits | per-minute and daily caps |
| T06-08 | Eval runner (`make eval`) | Tier 0–2 offline |
| T06-09 | Datasets v1 | grounding, scope, injection, quality, provider-compat |
| T06-10 | Report artefact + thresholds | `eval-report.json`, E-01..E-12 |
| T06-11 | Traceability matrix generator | FR → AC → test markers |
| T06-12 | CI job for eval on AI-touching changes | path-filtered |
| T06-13 | Optional judge harness | sampled, rubric, off by default |

## Exit criteria

- AC-GRD-01..05 and AC-EVAL-01..03 pass.
- Injection suite (≥ 20 cases) 100% resistant.
- Every FR has at least one test marker; matrix generated and committed.
- CI blocks AI-related PRs without an eval run.

## Out of scope

Production tracing/dashboards (Stage 07), live provider probing in CI.

## Risks

| Risk | Mitigation |
| --- | --- |
| Refusal false positives blocking legitimate analysis | two-sided dataset (in-scope + out-of-scope) |
| Eval flakiness with live models | offline recorded fixtures by default |
| Judge bias | rubric anchors, model separation, human spot-check |
