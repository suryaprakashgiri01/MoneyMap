# Non-Functional Requirements

**Document ID:** SPEC-REQ-002 · **Stage:** 00 · **Status:** Approved v1.0

Priority: **M** = MVP release blocker, **S** = should, **W** = won't (MVP).

## 1. Security (NFR-SEC)

| ID | Requirement | P |
| --- | --- | --- |
| NFR-SEC-01 | Passwords hashed with Argon2id (memory ≥ 19 MiB, iterations ≥ 2, parallelism 1 minimum) — no MD5/SHA/bcrypt-only schemes. | M |
| NFR-SEC-02 | Access tokens signed with an asymmetric or strong symmetric key from config; secret rotation supported without downtime (key id in header). | M |
| NFR-SEC-03 | All input validated with Pydantic v2 strict models; unknown fields rejected on auth and mutation endpoints. | M |
| NFR-SEC-04 | Parameterised ORM queries only. Raw SQL requires review and is limited to migrations/reporting. | M |
| NFR-SEC-05 | Security headers on all responses: `Strict-Transport-Security`, `X-Content-Type-Options`, `Referrer-Policy`, restrictive `Content-Security-Policy` on the web app. | M |
| NFR-SEC-06 | CORS restricted to configured origins; no wildcard with credentials. | M |
| NFR-SEC-07 | Dependency scanning (`pip-audit` / `npm audit`) and secret scanning run in CI. | M |
| NFR-SEC-08 | Rate limiting on auth (5/min/IP), compute (60/min/user), AI (10/min/user). | M |

## 2. Privacy & data protection (NFR-PRIV)

| ID | Requirement | P |
| --- | --- | --- |
| NFR-PRIV-01 | Financial amounts are never written to application logs, traces, or error messages. | M |
| NFR-PRIV-02 | LLM prompts contain only the minimum fields required for the specific result being explained. | M |
| NFR-PRIV-03 | Telemetry user identifiers are pseudonymous hashes; no email, phone, or name. | M |
| NFR-PRIV-04 | Account deletion hard-deletes profile, holdings, scenarios, memory and feedback within 30 days, with an immediate soft-delete. | M |
| NFR-PRIV-05 | Data export (JSON) of the user's own records available on request. | S |
| NFR-PRIV-06 | Provider data-retention posture documented; zero-retention/training opt-out used where the provider supports it. | M |
| NFR-PRIV-07 | No analytics/tracking third parties on authenticated pages in MVP. | M |

## 3. Correctness (NFR-COR)

| ID | Requirement | P |
| --- | --- | --- |
| NFR-COR-01 | Financial engine functions are pure, side-effect free, and clock-injected. | M |
| NFR-COR-02 | Engine branch coverage ≥ 95%; line coverage ≥ 90% for `domain/`. | M |
| NFR-COR-03 | Golden-number fixtures validate to exact `Decimal` equality; no `approx` comparisons for money. | M |
| NFR-COR-04 | Timezone-aware timestamps only (`Asia/Kolkata` for user-facing dates, UTC for storage). | M |
| NFR-COR-05 | Interest/date conventions (day-count, compounding, annuity-due vs ordinary) are documented per formula. | M |

## 4. Performance (NFR-PERF)

| ID | Requirement | P |
| --- | --- | --- |
| NFR-PERF-01 | Non-AI API p95 < 300 ms, p99 < 800 ms at MVP load. | M |
| NFR-PERF-02 | AI explanation p95 < 8 s; hard timeout at 20 s with graceful failure. | M |
| NFR-PERF-03 | Cold page load (LCP) < 2.5 s on a mid-tier mobile over 4G for the dashboard. | S |
| NFR-PERF-04 | DB indexes on all tenant + lookup paths used by list endpoints; no N+1 queries (asserted via query counter in tests). | M |

## 5. Reliability (NFR-REL)

| ID | Requirement | P |
| --- | --- | --- |
| NFR-REL-01 | Health endpoints: `/healthz` (liveness) and `/readyz` (DB + config check). | M |
| NFR-REL-02 | AI provider failure degrades to a deterministic "explanation unavailable" response; the engine result is still returned. | M |
| NFR-REL-03 | Retries only for transient classes (timeout, 429, 5xx) with bounded exponential backoff + jitter; never for 4xx. | M |
| NFR-REL-04 | Migrations are forward-only, reviewed, and reversible where feasible; app starts only if schema revision matches. | M |

## 6. AI provider independence (NFR-AI)

| ID | Requirement | P |
| --- | --- | --- |
| NFR-AI-01 | Business code depends on the `ModelGateway` protocol only; provider SDKs are confined to the adapter layer. | M |
| NFR-AI-02 | Model id, temperature, max tokens, timeout come from config; switching provider requires no code change. | M |
| NFR-AI-03 | A capability registry declares structured-output / tool-calling / streaming / context-limit / embedding support per model; unsupported features fail at config-validation time, not at runtime. | M |
| NFR-AI-04 | Adapters expose normalised errors (`rate_limited`, `timeout`, `auth_failed`, `invalid_request`, `provider_error`). | M |
| NFR-AI-05 | Each supported provider's limitations are documented in `docs/architecture/ai-workflow.md`. | S |

## 7. Maintainability (NFR-MAINT)

| ID | Requirement | P |
| --- | --- | --- |
| NFR-MAINT-01 | Python 3.12–3.13, managed by `uv`; lockfile committed. | M |
| NFR-MAINT-02 | `ruff` (lint + format) and `mypy --strict` on `domain/` and `ai/` gating CI. | M |
| NFR-MAINT-03 | Layering enforced by import rules: `api → services → domain/infra`; `domain` imports nothing from `api`/`infra`. | M |
| NFR-MAINT-04 | No module > 400 lines without justification; no function > 50 lines in `domain/`. | S |
| NFR-MAINT-05 | Public functions documented with docstrings stating formula source and assumptions. | M |

## 8. Accessibility & UX (NFR-UX)

| ID | Requirement | P |
| --- | --- | --- |
| NFR-UX-01 | WCAG 2.1 AA: keyboard navigable, visible focus, labels on all inputs, ≥ 4.5:1 text contrast. | M |
| NFR-UX-02 | All amounts rendered with currency symbol, thousand separators (Indian grouping) and period. | M |
| NFR-UX-03 | Every unfamiliar financial term has an inline definition (tooltip/expandable). | S |
| NFR-UX-04 | Empty, loading, error and `incomplete` states are designed, not defaulted. | M |
| NFR-UX-05 | Mobile-first responsive layout; no horizontal scroll for tables at 360 px (card fallback). | M |

## 9. Observability (NFR-OBS)

| ID | Requirement | P |
| --- | --- | --- |
| NFR-OBS-01 | OpenTelemetry traces for HTTP, DB, and AI spans; one trace per request. | M |
| NFR-OBS-02 | Structured JSON logs with `request_id`, `trace_id`, `route`, `status`, `duration_ms`. | M |
| NFR-OBS-03 | Langfuse captures prompt version, model, tokens, latency; redaction applied before export. | M |
| NFR-OBS-04 | Metrics: request rate, error rate, p95 latency, AI retry count, grounding-failure count. | M |

## 10. Testability (NFR-TEST)

| ID | Requirement | P |
| --- | --- | --- |
| NFR-TEST-01 | Test suite runs in one command with no external network by default (AI/provider mocked). | M |
| NFR-TEST-02 | API contract tests derive from the OpenAPI schema; schema drift fails CI. | M |
| NFR-TEST-03 | Every FR has at least one automated test reference (traceability matrix). | M |
| NFR-TEST-04 | Testcontainers or Compose Postgres for integration tests; SQLite is not acceptable for DB tests. | M |

## 11. Cost & operability (NFR-COST)

| ID | Requirement | P |
| --- | --- | --- |
| NFR-COST-01 | AI cost per explanation < USD 0.01 at default model config (short prompts, capped tokens). | M |
| NFR-COST-02 | Local dev runs fully via `docker compose up` with seeded synthetic data. | M |
| NFR-COST-03 | Public MVP runs on a single small instance + managed Postgres free/low tier. | M |
