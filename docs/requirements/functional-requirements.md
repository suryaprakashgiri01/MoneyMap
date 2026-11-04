# Functional Requirements

**Document ID:** SPEC-REQ-001 · **Stage:** 00 · **Status:** Approved v1.0

Priority: **M** = MUST (MVP release blocker), **S** = SHOULD (MVP if time), **W** = WON'T (MVP).
All numeric output must originate from the financial engine and carry a provenance tag
(`provided` | `sourced` | `assumed` | `calculated` | `generated`) — this is assumed for every row
below and not repeated.

## A. Authentication & account

| ID | Requirement | P | Traces |
| --- | --- | --- | --- |
| FR-AUTH-01 | Email + password signup; password ≥ 12 chars, Argon2id hash, normalised email uniqueness. Creates exactly one personal org. | M | US-A1 |
| FR-AUTH-02 | Login issues short-lived access JWT (≤15 min) + rotating refresh token (≤30 d), stored httpOnly; refresh token reuse revokes the family. | M | US-A2 |
| FR-AUTH-03 | All `/v1/*` endpoints except health, signup and login require a valid access token; failures return 401 with a stable error code. | M | US-A2 |
| FR-AUTH-04 | "Log out all devices" revokes every refresh token family for the user. | M | US-A3 |
| FR-AUTH-05 | Failed logins are rate limited per IP and per account (lockout escalation after N failures). | M | — |

## B. Financial profile

| ID | Requirement | P | Traces |
| --- | --- | --- | --- |
| FR-PROF-01 | User records monthly income: net amount, currency INR, income type (`salaried` \| `variable`). | M | US-B1 |
| FR-PROF-02 | For `variable` income, user supplies a 3–12 month history and the engine uses the median, explicitly labelled as such. | M | US-B2 |
| FR-PROF-03 | User records expense lines with category, amount, periodicity (`monthly` \| `annual` \| `one_off`) and essential/optional flag. | M | US-B3 |
| FR-PROF-04 | Annual and one-off expenses are normalised to monthly equivalents using documented rules (`months/12`, explicitly labelled). | M | US-B3 |
| FR-PROF-05 | Profile edits are versioned by `updated_at`; no field is destructive-overwritten without an audit row. | S | — |

## C. Derived metrics

| ID | Requirement | P | Traces |
| --- | --- | --- | --- |
| FR-METR-01 | `monthly_surplus = income_monthly − total_expense_monthly`; may be negative and must render as negative. | M | US-B4 |
| FR-METR-02 | `savings_ratio = surplus / income_monthly`; defined as `null` (not 0) when income is zero. | M | US-B4 |
| FR-METR-03 | `expense_ratio = total_expense_monthly / income_monthly`; `null` when income is zero. | M | US-B4 |
| FR-METR-04 | Ratios returned as basis points (integer) plus a display string rounded to 2 dp; raw `Decimal` retained. | M | US-B4 |

## D. Emergency fund

| ID | Requirement | P | Traces |
| --- | --- | --- | --- |
| FR-EMRG-01 | Target = `months × essential_expense_monthly`; `months` user-set, default 6, range 1–24. | M | US-C1 |
| FR-EMRG-02 | Engine returns target, current liquid assets, gap (target − current, floored at 0), and coverage months. | M | US-C1 |
| FR-EMRG-03 | Liquid asset classification is explicit per holding; non-liquid holdings never count toward coverage. | M | US-C2 |
| FR-EMRG-04 | If essential expenses are unclassified, the engine returns `incomplete` rather than assuming 100% or 0%. | M | US-C2 |

## E. Goals

| ID | Requirement | P | Traces |
| --- | --- | --- | --- |
| FR-GOAL-01 | Goal = name, target amount, target date (date-only, IST), optional linked holding ids, priority. | M | US-C3 |
| FR-GOAL-02 | Goal progress = allocated amount / target, `null` if no allocation exists. | M | US-C3 |
| FR-GOAL-03 | Required monthly contribution computed to hit target by date (rate = 0 or user rate, stated). | M | US-C4 |
| FR-GOAL-04 | If required contribution > surplus, engine flags `infeasible` with the shortfall amount. | M | US-C4 |

## F. Product catalogue & rates

| ID | Requirement | P | Traces |
| --- | --- | --- | --- |
| FR-CAT-01 | Catalogue covers `fd`, `rd`, `ppf`, `post_office_td`, `post_office_rd`, `post_office_mis`, `post_office_scss`, `mutual_fund_sip`, `lic_policy`. | M | US-D1 |
| FR-RATE-01 | Rates live in a versioned dataset with `value`, `effective_from`, `source_name`, `source_url`, `retrieved_at`, `verified_by`. | M | US-D2 |
| FR-RATE-02 | API exposes rate provenance; UI shows "as of" date and source for every quoted rate. | M | US-D2 |
| FR-RATE-03 | A calculation MUST refuse to run when the required rate is missing or unverified, returning `missing_input` with the field name. | M | US-E3 |
| FR-RATE-04 | A rate older than its staleness window is returned with `stale: true` and the comparison is marked `partial`. | M | US-E3 |

## G. Holdings

| ID | Requirement | P | Traces |
| --- | --- | --- | --- |
| FR-HOLD-01 | Holding = type, principal/contribution, start date, tenure, rate (sourced or user), optional maturity & current value. | M | US-D1 |
| FR-HOLD-02 | LIC holding additionally records: sum assured, annual premium, premium term, policy term, start date, bonus assumption. | M | US-D4 |
| FR-HOLD-03 | Holdings are editable and soft-deletable; deletion preserves an audit record. | M | US-D3 |
| FR-HOLD-04 | Every amount field is validated (`> 0`, ≤ 10^12, ≤ 2 dp) and rejected with a field-level error otherwise. | M | US-D1 |

## H. Deterministic calculations

| ID | Requirement | P | Traces |
| --- | --- | --- | --- |
| FR-CALC-01 | Future value of monthly contributions with monthly compounding: `FV = P·[((1+i)^n − 1)/i]·(1+i)`, `i = annual/12`, documented as annuity-due. | M | US-E2 |
| FR-CALC-02 | FD maturity with quarterly compounding: `A = P·(1 + r/4)^(4t)`; simple-interest fallback only if explicitly stated. | M | US-E2 |
| FR-CALC-03 | RD maturity via per-instalment quarterly compounding, one instalment maturity shaved per month. | M | US-E2 |
| FR-CALC-04 | PPF: annual deposits, annual compounding, 15-year term, extension handling documented; no premature-withdrawal modelling. | M | US-E2 |
| FR-CALC-05 | XIRR via Newton–Raphson with bisection fallback, tolerance 1e-9, max 100 iterations; non-convergence raises a typed error, never returns a guess. | M | US-D4 |
| FR-CALC-06 | All money values rounded with `ROUND_HALF_UP` to 2 dp only at the presentation boundary; intermediates keep full `Decimal` precision. | M | US-E2 |
| FR-CALC-07 | Each result carries `assumptions[]` and `formula_id` so the UI can show the derivation. | M | US-B5 |

## I. Comparison

| ID | Requirement | P | Traces |
| --- | --- | --- | --- |
| FR-CMP-01 | Comparison normalises instruments to identical horizon, contribution, and compounding disclosure. | M | US-E1 |
| FR-CMP-02 | Comparison rows include return, risk band, liquidity days, lock-in, tax treatment note, horizon fit, and insurance cover (where applicable). | M | US-E1 |
| FR-CMP-03 | Comparison never emits a single "best" ranking; it returns a metric matrix plus neutral deltas. | M | US-E1 |
| FR-CMP-04 | Any input that makes rows non-comparable is surfaced as `not_comparable` with the reason. | M | US-E3 |

## J. Scenarios

| ID | Requirement | P | Traces |
| --- | --- | --- | --- |
| FR-SCN-01 | Scenario = baseline snapshot + allocation changes (source holding, target instrument, monthly amount, start, duration). | M | US-F1 |
| FR-SCN-02 | Scenario output shows deterministic deltas per instrument and total, at the scenario horizon. | M | US-F2 |
| FR-SCN-03 | Scenarios are immutable snapshots; they never mutate holdings, goals, or profile. | M | US-F2 |
| FR-SCN-04 | Two scenarios can be diffed on the same normalised basis. | S | US-F3 |

## K. AI explanation

| ID | Requirement | P | Traces |
| --- | --- | --- | --- |
| FR-AI-01 | `POST /v1/insights/explain` accepts a `result_ref` (server-side computed result id) and returns a grounded explanation. | M | US-G1 |
| FR-AI-02 | The prompt receives only the structured result payload + minimal user context; it never receives credentials or unrelated holdings. | M | US-G2 |
| FR-AI-03 | Output validation rejects any numeric token not present in the result payload (grounding check) and retries once, then fails closed. | M | US-G3 |
| FR-AI-04 | All model calls go through `ModelGateway`; model id and prompt version come from configuration. | M | — |
| FR-AI-05 | When the engine result is `incomplete`, `partial`, or error, the AI returns a fixed non-numeric explanation of what is missing, without inventing values. | M | US-G3 |
| FR-AI-06 | AI responses include `disclaimer`, `prompt_version`, `model`, and `grounding: {checked: true, unmatched: 0}`. | M | US-G4 |

## L. Memory

| ID | Requirement | P | Traces |
| --- | --- | --- | --- |
| FR-MEM-01 | Memory entries have `scope` ∈ {`user`, `organization`, `generic`}, `kind`, `content`, `provenance`, `created_at`, `updated_at`. | M | US-H1 |
| FR-MEM-02 | User memory is written only on explicit user confirmation (a UI action), never auto-extracted from text. | M | US-H1 |
| FR-MEM-03 | Users can list and delete their own memory entries; deletion is immediate and irreversible. | M | US-H2 |
| FR-MEM-04 | Retrieval is scoped: user memory by `user_id`, org memory by `org_id`, generic by `published` flag only. | M | US-H3 |
| FR-MEM-05 | Secrets, credentials, and account numbers are rejected at write time by a validator. | M | — |

## M. Tenancy & isolation

| ID | Requirement | P | Traces |
| --- | --- | --- | --- |
| FR-TEN-01 | Every tenant-scoped table carries `org_id`; all repository queries filter by it; cross-org access returns 404, not 403. | M | US-H3 |
| FR-TEN-02 | Automated security tests attempt cross-org reads/writes on every tenant-scoped resource. | M | US-H3 |
| FR-TEN-03 | The AI context builder refuses to assemble context spanning more than one `org_id`. | M | US-H3 |

## N. Transparency

| ID | Requirement | P | Traces |
| --- | --- | --- | --- |
| FR-TRAN-01 | Every computed response includes inputs used, assumptions, formula ids, and timestamp. | M | US-B5 |
| FR-TRAN-02 | Assumption catalogue is served by `GET /v1/meta/assumptions` with human-readable text. | M | US-I1 |
| FR-TRAN-03 | `partial`/`incomplete`/`stale` states are first-class response fields, not warnings buried in text. | M | US-E3 |
| FR-TRAN-04 | Every response includes `trace_id` and `request_id`. | M | — |

## O. Guardrails & scope control

| ID | Requirement | P | Traces |
| --- | --- | --- | --- |
| FR-GRD-01 | Requests asking for transactions, guarantees, market timing, tax filing, or "which is best" are classified out-of-scope and refused with the canonical refusal text. | M | US-G3 |
| FR-GRD-02 | Retrieved and user-supplied text is treated as data; instruction-like content never changes system behaviour. | M | — |
| FR-GRD-03 | AI output is scanned for forbidden language patterns from `non-goals-and-disclaimers.md` §4 before return. | M | US-G4 |
| FR-GRD-04 | Rate limits: per-user and per-IP on auth, compute, and AI endpoints, with 429 + `Retry-After`. | M | — |
| FR-GRD-05 | The AI may not call any mutating tool: the tool set exposed to the model is read-only and allow-listed. | M | — |

## P. Disclaimers & feedback

| ID | Requirement | P | Traces |
| --- | --- | --- | --- |
| FR-DISC-01 | Server injects the applicable disclaimer(s) into every AI and comparison/scenario payload. | M | US-G4 |
| FR-FDBK-01 | Users can rate an explanation (useful / not useful / inaccurate) with optional comment. | S | US-I2 |
| FR-FDBK-02 | Feedback is stored without raw financial values, linked by `trace_id`, and can be exported to the eval dataset. | S | US-I2 |

## Q. Observability

| ID | Requirement | P | Traces |
| --- | --- | --- | --- |
| FR-OBS-01 | Every request has a `request_id` propagated into logs, spans and AI traces. | M | — |
| FR-OBS-02 | LLM traces record provider, model, prompt version, latency, token usage, retry count, and eval flags — no raw amounts. | M | — |
| FR-OBS-03 | User identity in telemetry is a stable pseudonymous hash, never email or name. | M | — |

## R. Evaluation

| ID | Requirement | P | Traces |
| --- | --- | --- | --- |
| FR-EVAL-01 | A versioned eval dataset and runner exist; failures are reproducible locally via one command. | M | — |
| FR-EVAL-02 | Deterministic assertions (schema, grounding, forbidden patterns, golden numbers) run before any LLM-judge scoring. | M | — |
| FR-EVAL-03 | Golden financial cases are stored as fixtures and re-run in CI on every prompt/model/engine change. | M | — |
