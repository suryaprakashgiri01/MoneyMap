# ADR-0011 — Tax Treatment as Labelled Notes; No Slab Engine

**Status:** Accepted · **Date:** 2026-11-04 · **Stage:** 00

## Context

Tax is the most requested and most dangerous comparison axis. A real tax engine needs regime choice,
slab arithmetic, surcharge, cess, deduction caps, age bands, and annual legal changes. Getting it
subtly wrong produces confident, wrong advice-adjacent output. The product is explicitly not a tax
adviser (`non-goals-and-disclaimers.md`).

## Decision

1. Comparisons include a **tax treatment note** per instrument: taxable/EEE/EET, section reference
   (e.g. 80C, 10(10D)), TDS applicability, and a one-line caveat.
2. No slab computation, no regime comparison, no deduction optimisation, no ITR export.
3. Tax notes are static, reviewed content (versioned like rates) — not model-generated.
4. If a notional "post-tax" figure is shown, it is computed with a single user-declared marginal rate
   that the user must explicitly enter; the response marks it `assumed` and shows the rate used.
5. Any user request for tax computation is refused via the static refusal table.

## Consequences

- We under-serve users who want tax optimisation — acceptable and documented.
- Comparisons remain legally safer and mathematically simpler.
- Adding a slab engine later is possible behind its own ADR, with a test corpus and a legal review.

## Alternatives considered

- **Full slab engine** — rejected for MVP: legal volatility, high error cost, no requirement.
- **Ask the LLM for tax treatment** — rejected: hallucination risk on a regulated topic.
- **Omit tax entirely** — rejected: tax treatment materially changes product comparisons.

## Compliance

- FR-CMP-02, FR-GRD-01, `product-scope.md` §3.10.
