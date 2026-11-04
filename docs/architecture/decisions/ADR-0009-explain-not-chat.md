# ADR-0009 — AI Explains Computed Results; No Free-Form Chat

**Status:** Accepted · **Date:** 2026-11-04 · **Stage:** 00

## Context

A general financial chatbot is attractive for demos and corrosive for correctness: it invites
free-form numeric answers, requires conversation storage, expands the prompt-injection surface, and
makes grounding nearly impossible to enforce. The product's differentiator is *auditable numbers*.

## Decision

1. The only AI endpoint in MVP is `POST /v1/insights/explain` with a `result_ref` pointing at a
   server-persisted `computed_results` row.
2. The request cannot carry amounts, descriptions, or free text. Input is `result_ref`, `style`,
   `locale`.
3. There is no conversation state, no message history, no transcript storage.
4. Out-of-scope questions are answered by a static refusal table, without a model call.
5. If a future requirement needs Q&A, it must be added as a *new* endpoint with its own grounding
   strategy and an ADR superseding this one.

## Consequences

- Grounding is provable: allowed numbers = payload numbers.
- Prompt-injection surface is tiny (no user text reaches the model except `notes` fields, which are
  fenced as data and never in the system prompt).
- Lower cost and latency: one call, ~1,200 input tokens.
- UX limitation: users cannot "ask follow-ups" freely; they adjust inputs and re-run a result.
  Accepted for MVP because it forces the user back into the deterministic model.

## Alternatives considered

- **Full assistant with tool calling** — rejected for MVP: unbounded scope, hard evaluation, higher
  cost, larger injection surface.
- **Chat with retrieval over user's own notes** — rejected: PII retention and unverifiable claims.
- **Chat plus mandatory numeric post-check** — rejected: post-checks cannot resolve *which* number
  was intended when the model computes something new.

## Compliance

- FR-AI-01..06, SPEC-ARCH-003, AC-AI-01..06.
