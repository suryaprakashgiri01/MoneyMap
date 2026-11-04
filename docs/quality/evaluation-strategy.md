# Evaluation Strategy

**Document ID:** SPEC-QLT-002 · **Stage:** 00 · **Status:** Approved v1.0
**Related:** FR-EVAL-01..03, T-01..T-10

## 1. Objective

Prove, repeatably and cheaply, that MoneyMap's AI layer (a) never fabricates numbers, (b) stays in
scope, (c) resists injection, and (d) actually explains things usefully — without paying for an LLM
judge on every assertion.

## 2. Ordering rule

**Deterministic assertions run first and can fail a release on their own.** An LLM judge is only used
for qualities that cannot be asserted mechanically (clarity, usefulness, tone), and never as the sole
evidence for a correctness claim.

```
Tier 0: schema + grounding + forbidden patterns + refusal matching   (deterministic, offline)
Tier 1: golden financial numbers vs engine                            (deterministic, offline)
Tier 2: retrieval/context correctness                                 (deterministic, offline)
Tier 3: judge-scored usefulness/clarity                               (LLM judge, opt-in, sampled)
```

## 3. Datasets (versioned, committed)

`backend/evals/datasets/`

| File | Cases | Purpose |
| --- | --- | --- |
| `grounding_v1.jsonl` | 40 | payload + candidate explanation pairs (pass/fail) |
| `scope_v1.jsonl` | 40 | in-scope vs out-of-scope requests + expected refusal code |
| `injection_v1.jsonl` | 25 | injection payloads in notes/memory; expected unchanged behaviour |
| `explain_quality_v1.jsonl` | 20 | real-ish payloads with rubric for judge (Tier 3) |
| `provider_compat_v1.jsonl` | 12 | same request across provider profiles |
| `financial_golden_v1.json` | 30 | engine formula cases (reused by unit tests) |

Rules:
- Append-only with new versions (`_v2`), never edit an old version in place.
- Every case has `id`, `input`, `expect`, `tags`, `added_in_stage`.
- No real user data; payloads are synthetic and reviewed.

## 4. Metrics and thresholds

| ID | Metric | Tier | Threshold | Blocks release |
| --- | --- | --- | --- | --- |
| E-01 | Grounding precision (unmatched numbers per response) | 0 | 0 | yes |
| E-02 | Schema validity | 0 | ≥ 99.5% | yes |
| E-03 | Forbidden-language violations | 0 | 0 | yes |
| E-04 | Refusal correctness (in/out of scope) | 0 | ≥ 98% | yes |
| E-05 | Injection resistance | 0 | 100% | yes |
| E-06 | Golden financial exactness | 1 | 100% | yes |
| E-07 | Context correctness (right memory, nothing foreign) | 2 | 100% | yes |
| E-08 | Judge usefulness (1–5) | 3 | mean ≥ 3.8 | no (report) |
| E-09 | Judge clarity/factual tone | 3 | mean ≥ 4.0 | no |
| E-10 | Latency p95 | 0 | ≤ 8 s | no (alarm) |
| E-11 | Cost per explanation | 0 | ≤ USD 0.01 | no (alarm) |
| E-12 | Provider compatibility pass rate | 0 | 100% of required capabilities | yes |

## 5. Runner

```
make eval                 # Tier 0–2 offline with FakeGateway + recorded fixtures
make eval-judge           # Tier 3, requires provider key, sampled, cost-capped
make eval-provider PROVIDER=groq   # optional live compatibility probe
```

- Emits `eval-report.json` + human summary, stored as a CI artefact and referenced in the PR.
- Deterministic mode uses recorded provider responses so results are reproducible offline.
- Each run records: `git_sha`, `prompt_version`, `model_id`, `dataset_version`, `engine_version`.

## 6. Judge design (Tier 3 only)

- Judge model is configured separately (`EVAL_JUDGE_MODEL`) and must be different from the served model.
- Rubric is explicit, 1–5, with anchors and 3 few-shot examples.
- Judges see: payload, explanation, rubric. They do **not** see provider/model identity.
- Limit: judge scores never override a Tier-0 failure; a passing judge does not rescue a grounding error.
- Spot-check: 10% of judge-scored outputs are human-reviewed during the stage that introduces them.

## 7. Regression policy

| Change | Required evals |
| --- | --- |
| Prompt template edit | Tier 0 + 1 + 3, full dataset |
| Model/provider change | Tier 0 + 1 + provider compat, plus Tier 3 sample |
| Engine formula change | Tier 1 full + unit/property suite |
| Memory retrieval change | Tier 2 full |
| Guardrail change | Tier 0 scope + injection full |
| Unrelated UI change | Tier 0 smoke (10 cases) |

Any threshold regression blocks the PR unless an ADR documents the accepted trade-off.

## 8. Failure taxonomy (for triage and metrics)

`grounding_miss`, `schema_invalid`, `forbidden_language`, `scope_leak`, `refusal_false_positive`,
`injection_success`, `context_leak`, `provider_error`, `timeout`, `cost_overrun`, `judge_low_score`.

Counts per taxonomy bucket are exported as metrics (Stage 07) so regressions are visible in production.

## 9. Production feedback loop

1. User feedback (`feedback_entries`) references `trace_id` + `prompt_version`.
2. Weekly triage: sample negative feedback, classify into the taxonomy.
3. Confirmed failures become new dataset cases (append-only) with a regression test.
4. Prompt/model changes follow the human-reviewed PR process — no autonomous prompt updates.

## 10. Explicit non-goals

- No continuous online training or self-modifying prompts.
- No A/B testing infrastructure in MVP (single variant, versioned).
- No judge-only gating of releases.
- No synthetic user simulation at scale.
