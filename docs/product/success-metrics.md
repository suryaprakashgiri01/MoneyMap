# Success Metrics

**Document ID:** SPEC-PROD-005 · **Stage:** 00 · **Status:** Approved v1.0

Two classes of metrics: **product value** (does it help) and **system trustworthiness** (is it
correct, honest and safe). Trust metrics are release-blocking; value metrics are directional.

## 1. Product value metrics (MVP targets)

| ID | Metric | Definition | MVP target |
| --- | --- | --- | --- |
| M-01 | Activation | New user completes profile with income + ≥3 expense lines | ≥ 60% of signups |
| M-02 | Time to first insight | Signup → first rendered savings ratio | median ≤ 3 min |
| M-03 | Holdings captured | Users with ≥1 holding recorded within 24h | ≥ 40% |
| M-04 | Comparison engagement | Users who open a comparison table | ≥ 50% of activated |
| M-05 | Scenario completion | Users who save ≥1 scenario | ≥ 25% of activated |
| M-06 | Explanation usefulness | Feedback "useful" on AI explanation | ≥ 75% of ratings |
| M-07 | 7-day return | Users returning within 7 days | ≥ 30% (directional) |

## 2. Correctness & trust metrics (release-blocking)

| ID | Metric | Definition | Threshold |
| --- | --- | --- | --- |
| T-01 | Engine determinism | Identical inputs → byte-identical `Decimal` output across 1,000 runs | 100% |
| T-02 | Golden-case accuracy | Engine results vs hand-computed golden file | 100% exact, 0 tolerance |
| T-03 | Explanation grounding | Numeric tokens in AI text that match engine output | 100% (zero unmatched numbers) |
| T-04 | Structured output validity | AI responses passing schema validation | ≥ 99.5% |
| T-05 | Hallucinated product rate | Invented rates in AI text | 0 occurrences |
| T-06 | Scope adherence | AI responses correctly refusing out-of-scope asks | ≥ 98% on eval set |
| T-07 | Prompt-injection resistance | Injected instructions that alter behaviour | 0 occurrences |
| T-08 | Tenant isolation | Cross-org data leakage in security tests | 0 occurrences |
| T-09 | Provenance completeness | Computed fields carrying a provenance tag | 100% |
| T-10 | Missing-data honesty | Responses inventing a value for a missing input | 0 occurrences |

## 3. Operational metrics

| ID | Metric | Target (MVP) |
| --- | --- | --- |
| O-01 | API p95 latency (non-AI) | < 300 ms |
| O-02 | AI explanation p95 latency | < 8 s |
| O-03 | Error rate (5xx) | < 1% |
| O-04 | AI cost per explanation | < USD 0.01 |
| O-05 | Availability (monthly) | ≥ 99% (best-effort MVP) |
| O-06 | Trace coverage | 100% of AI calls traced with request ID, model, prompt version |

## 4. Evaluation gates

A release is blocked if any of the following regress against the previous tagged evaluation run:

- T-02 golden cases, T-03 grounding, T-05 hallucination, T-07 injection, T-08 isolation.
- Overall eval score drop > 2 percentage points on any suite in `docs/quality/evaluation-strategy.md`.

## 5. Anti-metrics (things we refuse to optimise)

- Session length / engagement time — the product succeeds when the user leaves with an answer.
- Number of AI messages — more AI is not better.
- Products "recommended" — we count comparisons, not suggestions.
- Conversions to any financial product — no affiliate or referral mechanics in MVP.
