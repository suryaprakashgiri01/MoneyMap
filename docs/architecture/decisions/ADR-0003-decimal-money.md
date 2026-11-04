# ADR-0003 — Decimal Money and Rounding Discipline

**Status:** Accepted · **Date:** 2026-11-04 · **Stage:** 00

## Context

Binary floating point cannot represent ₹0.10 exactly. A comparison table that is off by a paisa
undermines trust, and finance users *do* check arithmetic. Requirements forbid floats in domain code
(FR-CALC-06, NFR-COR-03).

## Decision

1. All monetary values are `decimal.Decimal`.
2. Storage: PostgreSQL `NUMERIC(18,2)`. Rates: `NUMERIC(9,6)` (decimal fraction, e.g. `0.071000`).
3. Intermediates keep full precision (context `decimal.Context(prec=28)`); rounding happens **only**
   at the presentation boundary.
4. Rounding mode: `ROUND_HALF_UP` to 2 dp for money; percentages displayed to 2 dp from basis points.
5. Ratios are computed as `Decimal` and exposed as integer basis points (`2941` = 29.41%) to avoid
   ambiguity between `0.2941` and `29.41`.
6. JSON serialisation uses strings for money (`"25000.00"`), never JSON numbers, so clients cannot
   lose precision.
7. A CI rule (ruff/mypy + AST check) forbids `float(` and `: float` annotations inside `domain/`.

## Consequences

- Exact golden-number tests are possible (AC-CALC-*, T-02).
- Slightly more verbose code and mandatory string parsing in DTOs.
- Frontend must format strings with `Intl.NumberFormat('en-IN')` rather than doing arithmetic.

## Alternatives considered

- **Integer paise** — rejected: interest math needs sub-paisa intermediates; conversion noise at every
  boundary.
- **`float` with epsilon comparisons** — rejected outright for money.
- **`money` third-party library** — rejected: extra dependency for a rule we can enforce in 20 lines.

## Compliance

- `domain/money.py` is the only place that quantises.
- Rounding rules documented in `docs/quality/test-strategy.md` §4.
