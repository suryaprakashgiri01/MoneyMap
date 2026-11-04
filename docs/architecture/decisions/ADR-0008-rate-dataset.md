# ADR-0008 — Versioned, Source-Cited Rate Dataset

**Status:** Accepted · **Date:** 2026-11-04 · **Stage:** 00

## Context

Comparisons depend on interest rates for FD, RD, PPF, Post Office schemes, and on assumptions for
equity SIP returns. Rates change (repo-linked FD rates, quarterly PPF resets). Scraping bank sites is
brittle, legally murky, and untestable. A hardcoded constant in code is worse: it silently rots.

## Decision

1. Rates live in `product_rates` rows (append-only) loaded from
   `backend/infra/seed/product_rates.yaml`.
2. Every row carries `value`, `effective_from`/`effective_to`, `source_name`, `source_url`,
   `retrieved_at`, `verified_by`, `is_assumption`.
3. Equity/market-linked expected returns are always `is_assumption = true` and rendered as
   "illustrative assumption", never as a product rate.
4. A calculation refuses to run without a verified, non-stale rate (FR-RATE-03/04): HTTP 422
   `missing_input` with the field path. No default fallback, ever.
5. Staleness window per product type in config (e.g. FD/RD 180 days, PPF 365 days, Post Office 365
   days). Stale rows are returned with `stale: true` and the response is `partial`.
6. Users may override a rate on their own holding (`rate_source = user`); the override is labelled
   `provided` and never silently replaces a sourced rate for comparisons.
7. Update cadence: a documented monthly manual review task; each refresh is a seed commit with a
   dated changelog entry. No runtime scraping in MVP.

## Consequences

- Every displayed rate is auditable and dated.
- Some comparisons will be unavailable rather than wrong — an intentional trade-off.
- Maintenance burden: a human must re-verify rates periodically (documented in Stage 09 runbook).

## Alternatives considered

- **Live scraping** — rejected: brittle parsers, ToS risk, no test stability.
- **Third-party rate API** — deferred: paid dependency, coverage gaps for Post Office schemes.
- **Hardcoded constants** — rejected: unauditable and silently stale.

## Compliance

- FR-RATE-01..04, FR-CAT-01, AC-RATE-01..04, AC-CMP-02.
