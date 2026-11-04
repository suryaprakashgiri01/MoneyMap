# ADR-0002 — Modular Monolith over Microservices

**Status:** Accepted · **Date:** 2026-11-04 · **Stage:** 00

## Context

MVP scope is ~15 domain concepts, one client, one team (one engineer + agent), < 1,000 users. Candidate
architectures ranged from a single FastAPI app to separate calculation, AI and identity services.

## Decision

Single deployable FastAPI application with enforced module boundaries:

```
api/ → services/ → domain/  (pure)
services/ → infra/
ai/ → domain/ (types), ai/adapters/ → LiteLLM
```

Boundaries are enforced by a CI import-graph test, not by process isolation. The financial engine is
pure Python with no framework imports so it can be extracted later without a rewrite.

## Consequences

- One release artefact, one migration stream, one log/trace pipeline.
- Transactional consistency is free (single DB, single process).
- Scaling is vertical until Redis-backed rate limiting is added.
- Risk: boundary erosion inside one process. Mitigated by import tests and review checklist.

## Alternatives considered

- **Microservices (calc / AI / identity)** — rejected: no independent scaling need, adds network
  failure modes, distributed tracing overhead, and duplicated tooling.
- **Serverless functions per endpoint** — rejected: cold starts hurt AI latency budget; local dev
  parity is worse.
- **Desktop/local-only app** — rejected: S15 requires a public deployment.

## Compliance

- Failure of the import-graph test blocks merge.
- Extracting a service later requires a new ADR superseding this one.
