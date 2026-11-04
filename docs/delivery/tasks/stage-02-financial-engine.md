# Stage 02 — Financial Engine

**Branch:** `stage/02-financial-engine`
**Spec:** FR-CALC-01..07, NFR-COR-01..05, ADR-0003, ADR-0004, SPEC-QLT-001 §3–4

## Objective

A pure, dependency-free, exhaustively tested financial engine. This is the product's correctness moat.

## Tasks

| ID | Task | Notes |
| --- | --- | --- |
| T02-01 | `domain/money.py` | Decimal type, quantise, formatting, bps helpers |
| T02-02 | `domain/result.py` | `Computed[T]` with provenance, formula_id, assumptions |
| T02-03 | Savings metrics | surplus, savings ratio, expense ratio (null-safe) |
| T02-04 | Expense normalisation | monthly/annual/one-off rules, documented |
| T02-05 | Emergency fund | target, gap, coverage, `incomplete` semantics |
| T02-06 | Goal projections | required monthly contribution, feasibility |
| T02-07 | Future value (annuity-due) | documented formula, zero-rate edge case |
| T02-08 | FD maturity | quarterly compounding, TDS-not-modelled note |
| T02-09 | RD maturity | per-instalment compounding, month-end clamping |
| T02-10 | PPF projection | annual deposit, annual compounding, 15-year term |
| T02-11 | XIRR | Newton–Raphson + bisection fallback, typed non-convergence error |
| T02-12 | LIC analysis | XIRR, cover-per-rupee, opportunity cost |
| T02-13 | Comparison matrix | normalisation, `not_comparable`, no ranking |
| T02-14 | Golden fixtures | 30 cases, exact Decimal equality |
| T02-15 | Property tests | monotonicity, determinism, boundaries |
| T02-16 | Formula registry + coverage check | CI verifies every formula has fixtures |
| T02-17 | Float ban enforcement | AST/ruff rule for `domain/` |

## Exit criteria

- All AC-CALC-* pass; coverage: `domain/` line ≥ 90%, branch ≥ 95%.
- Engine has zero imports from FastAPI, SQLAlchemy, HTTP, or AI layers.
- 1,000-run determinism test green.
- Rounding and date conventions documented in the module docstrings.

## Out of scope

Persistence, HTTP endpoints, AI.

## Risks

| Risk | Mitigation |
| --- | --- |
| Silent formula convention mismatch (annuity-due vs ordinary) | state convention in fixture + docstring + test name |
| XIRR non-convergence on pathological cash flows | bisection fallback + typed error + tests |
| Rounding drift in long projections | round only at the boundary; assert intermediates in tests |
