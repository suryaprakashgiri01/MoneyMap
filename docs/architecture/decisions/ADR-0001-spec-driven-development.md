# ADR-0001 — Spec-Driven Development and Branching Workflow

**Status:** Accepted · **Date:** 2026-11-04 · **Stage:** 00

## Context

MoneyMap handles financial data and AI-generated explanations. Unspecified changes create two risks:
silent scope expansion and financial claims that cannot be traced to a requirement. The project is
built in stages with an AI agent that must not invent behaviour.

## Decision

1. Every feature starts as a specification in `docs/` with stable IDs (`FR-*`, `NFR-*`, `AC-*`, `ADR-*`).
2. Implementation tasks reference those IDs in code comments only where non-obvious, and always in the
   stage task file and PR description.
3. One branch per stage (`stage/NN-name`), one logical commit per change cluster, conventional commits.
4. A stage cannot merge without its Definition of Done (`docs/quality/definition-of-done.md`) satisfied.
5. Ambiguities are recorded in the spec's "Open questions" section; material ones trigger an ADR.

## Consequences

- Slightly slower start per feature; dramatically cheaper rework and review.
- Traceability matrix must be maintained (cheap: one line per FR).
- Docs are part of the deliverable and reviewed like code.

## Alternatives considered

- **Code-first with docs later** — rejected: financial semantics drift is expensive to reverse.
- **One giant PRD** — rejected: hard to diff, hard to trace, encourages scope creep.

## Compliance

- `AGENTS.md` §2 binds agents to this workflow.
- Stage task files live in `docs/delivery/tasks/`.
