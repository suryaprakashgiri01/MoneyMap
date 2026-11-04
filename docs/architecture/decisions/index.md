# Architecture Decision Records

**Document ID:** SPEC-ARCH-005 · **Stage:** 00

| ADR | Title | Status | Stage |
| --- | --- | --- | --- |
| [0001](ADR-0001-spec-driven-development.md) | Spec-driven development and branching workflow | Accepted | 00 |
| [0002](ADR-0002-modular-monolith.md) | Modular monolith over microservices | Accepted | 00 |
| [0003](ADR-0003-decimal-money.md) | Decimal money and rounding discipline | Accepted | 00 |
| [0004](ADR-0004-deterministic-first.md) | Deterministic engine is the only source of numbers | Accepted | 00 |
| [0005](ADR-0005-model-gateway.md) | ModelGateway abstraction with LiteLLM adapter | Accepted | 00 |
| [0006](ADR-0006-no-multi-agent.md) | Custom linear orchestration; no agents or LangGraph | Accepted | 00 |
| [0007](ADR-0007-structured-memory.md) | Structured memory first; pgvector deferred | Accepted | 00 |
| [0008](ADR-0008-rate-dataset.md) | Versioned, source-cited rate dataset | Accepted | 00 |
| [0009](ADR-0009-explain-not-chat.md) | AI explains computed results; no free-form chat | Accepted | 00 |
| [0010](ADR-0010-org-tenancy.md) | Org-scoped tenancy with personal orgs at MVP | Accepted | 00 |
| [0011](ADR-0011-tax-notes-only.md) | Tax treatment as labelled notes; no slab engine | Accepted | 00 |
| [0012](ADR-0012-auth-session.md) | JWT access + rotating refresh tokens, Argon2id | Accepted | 00 |
| [0013](ADR-0013-observability.md) | OTel + Langfuse with privacy-safe redaction | Accepted | 00 |
| [0014](ADR-0014-deployment.md) | Single-container backend + managed Postgres + Vercel | Accepted | 00 |

## Status meanings

- **Proposed** — under discussion, no code may depend on it.
- **Accepted** — binding; implementation must comply.
- **Superseded** — replaced by a later ADR (link added, never deleted).
- **Deprecated** — no longer recommended, still present in code.

## Adding an ADR

1. Copy the template structure (Context / Decision / Consequences / Alternatives / Compliance).
2. Number sequentially; never renumber.
3. Update this table and the stage task file that implements it.
