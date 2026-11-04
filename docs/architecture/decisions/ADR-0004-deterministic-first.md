# ADR-0004 — Deterministic Engine is the Only Source of Numbers

**Status:** Accepted · **Date:** 2026-11-04 · **Stage:** 00

## Context

LLMs produce plausible arithmetic with no error signal. Financial projections (XIRR, maturity values,
comparisons) are the product's core value and must be auditable. Regulators and users both punish
unverifiable numbers.

## Decision

1. Every number a user sees is produced by `domain/` functions, then persisted in `computed_results`
   with `formula_ids`, `engine_version`, and provenance tags.
2. The AI path may only read `computed_results` by `result_ref`; it cannot receive client-supplied
   amounts, and it cannot query arbitrary tables.
3. Client input to the AI endpoint is limited to `result_ref`, `style`, `locale`.
4. Every numeric token in AI output must exist in the payload (grounding check, SPEC-ARCH-003 §4).
   Failure → retry once → fail closed with `status: unavailable`.
5. Deterministic endpoints never call a model.

## Consequences

- AI failure degrades gracefully (numbers still served).
- Slight UX cost: explanation is tied to a persisted result (24 h TTL).
- Payloads must include display strings so the model can quote exactly.

## Alternatives considered

- **Let the model compute and show its work** — rejected: unverifiable, unreproducible.
- **Let the model call a calculator tool** — rejected for MVP: adds a tool-permission surface and
  still requires grounding; frozen result refs are simpler and stronger.
- **Post-hoc numeric reconciliation only** — rejected: reconciliation without a source of truth is
  not possible.

## Compliance

- AC-AI-01, AC-CALC-01..07, T-03, T-05.
- Code review must reject any AI prompt that asks the model to calculate.
