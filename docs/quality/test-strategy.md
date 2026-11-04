# Test Strategy

**Document ID:** SPEC-QLT-001 · **Stage:** 00 · **Status:** Approved v1.0
**Related:** NFR-TEST-01..04, NFR-COR-01..05, AC-*

## 1. Objectives

1. Prove financial correctness exactly (no "approximately right" money).
2. Prove tenant isolation and auth behaviour adversarially.
3. Prove AI output never introduces a number the engine did not produce.
4. Keep the suite fast, offline by default, and deterministic.

## 2. Test layers

| Layer | Scope | Tooling | Speed budget |
| --- | --- | --- | --- |
| L1 Unit | `domain/` formulas, money, validators | pytest | < 5 s total |
| L2 Unit | `ai/` grounding, guardrails, context builder | pytest + FakeGateway | < 5 s |
| L3 Integration | services + repos + real Postgres | pytest-asyncio + testcontainers/Compose | < 90 s |
| L4 Contract | HTTP surface vs OpenAPI schema | httpx AsyncClient + schemathesis (subset) | < 60 s |
| L5 Security | auth, isolation matrix, injection, PII scan | pytest | < 30 s |
| L6 E2E (smoke) | signup → profile → metrics → explain | Playwright (Stage 08) | < 2 min |
| L7 Eval | AI quality suites | custom runner (`make eval`) | < 5 min offline |

Default `make test` runs L1–L5. `make test-e2e` and `make eval` are separate.

## 3. Financial test approach

### 3.1 Golden fixtures

`backend/tests/fixtures/golden/*.json`:

```json
{
  "case_id": "fd-3y-710bps",
  "formula_id": "fd.maturity.quarterly",
  "inputs": { "principal": "100000.00", "annual_rate": "0.071000", "years": 3 },
  "expected": { "maturity": "123439.29" },
  "assumptions": ["quarterly compounding", "no TDS deducted"],
  "derived_by": "hand calculation, reviewed"
}
```

Rules:
- Money compared as exact `Decimal` strings — never `pytest.approx`.
- Each fixture names the formula id and the assumptions used.
- Fixtures are append-only; changing an expected value requires a documented formula change + ADR note.

### 3.2 Property tests (Hypothesis)

| Property | Example |
| --- | --- |
| Monotonicity | Higher rate ⇒ maturity ≥ at lower rate |
| Non-negativity | Maturity ≥ principal for positive rate and term |
| Determinism | Same input twice ⇒ identical output |
| Round-trip | Sum of monthly contributions × months = total contributed (no drift) |
| Boundary | `months=1`, `rate=0`, `principal=0.01`, max bounds |

### 3.3 Formula coverage matrix

Every formula id in `domain/financial/formulas.py` must appear in at least one golden fixture and one
property test. A CI check compares the registry to the fixture index.

## 4. Rounding and comparison rules

- Intermediates: full `Decimal` precision (context prec=28).
- Boundary: `ROUND_HALF_UP`, 2 dp.
- Ratios: integer basis points; tests assert bps, not percentages.
- Date conventions: monthly periods mean "same day next month, clamped to month end"; tests cover
  31st-of-month and leap-year cases.
- Timezone: all fixtures use `Asia/Kolkata` for user dates, UTC for timestamps; a test asserts no naive
  datetimes are produced.

## 5. API contract tests

- Generated from the FastAPI OpenAPI schema at test time; a schema diff fails CI.
- Each endpoint has: happy path, validation error, auth-missing, auth-expired, foreign-org (404).
- Error bodies asserted for stable codes, not messages.

## 6. Security tests

1. **Isolation matrix**: for each tenant resource, user B requests each of A's ids (GET/PATCH/DELETE)
   → 404, empty body, `security_events` row written.
2. **Auth**: expired access token, reused refresh token (family revoked), lockout after N failures.
3. **Input**: mass-assignment attempts, oversized fields, negative/zero amounts, 3-decimal amounts,
   extreme dates.
4. **Injection**: 20+ prompt-injection payloads in `notes`, memory content, and (future) retrieved docs.
5. **PII scan**: full-flow log capture asserted free of amounts/emails; exported spans asserted free of
   currency patterns.

## 7. AI tests

| Test | Type |
| --- | --- |
| Grounding accepts exact payload numbers | deterministic |
| Grounding rejects a number not in payload | deterministic |
| Grouping/percent/lakh normalisation | unit, table-driven |
| Refusal table for out-of-scope intents | deterministic |
| Forbidden-language scan | deterministic |
| Provider switch by config | integration with two fake adapters |
| Capability validation at startup | unit |
| Retry only on transient errors | unit with scripted fake |
| Timeout → `unavailable` + engine result intact | integration |
| Quality/grounding score | eval suite (SPEC-QLT-002) |

## 8. Test data policy

- Synthetic only. Names like "Test User A"; incomes like ₹85,000.
- Fixtures never contain real account numbers, PAN, phone numbers, or emails of real people.
- Seeded users are created by a fixture factory with deterministic ids where useful.

## 9. Coverage gates

| Area | Line | Branch |
| --- | --- | --- |
| `domain/` | ≥ 90% | ≥ 95% |
| `ai/` (validation/grounding/guardrails) | ≥ 90% | ≥ 90% |
| `services/` | ≥ 80% | ≥ 75% |
| `api/` | ≥ 75% | — |
| Overall | ≥ 80% | — |

Coverage is a floor, not a goal; missing critical-path tests block merge regardless of percentage.

## 10. CI pipeline

```
lint (ruff, mypy) → unit (L1,L2) → integration (L3, Postgres service)
  → contract (L4) → security (L5) → golden-fixtures check → coverage gate
  → (on prompt/model/engine change) eval offline suite
  → build image (on main/stage branches)
```

- Fail fast; parallel where independent.
- No network egress required except the Postgres service container.
- Artefacts: coverage report, eval report JSON, junit XML.

## 11. Traceability matrix

`docs/quality/traceability.md` (generated during Stage 06) maps `FR-* → AC-* → test ids`. A CI check
fails if an FR has no referenced test marker (`@pytest.mark.fr("FR-...")`).

## 12. Non-goals

- No load testing beyond a single k6 smoke script (Stage 09).
- No browser cross-matrix testing (Chromium + one mobile viewport only).
- No mutation testing in MVP.
