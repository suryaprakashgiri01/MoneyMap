# System Architecture

**Document ID:** SPEC-ARCH-001 · **Stage:** 00 · **Status:** Approved v1.0
**Related:** ADR-0002 (monolith), ADR-0005 (gateway), ADR-0006 (orchestration), ADR-0010 (tenancy), ADR-0013 (observability), ADR-0014 (deployment)

## 1. Architectural style

A **modular monolith**: one deployable FastAPI application, one Next.js frontend, one PostgreSQL
database. Modules are separated by directory and enforced by import rules, not by network calls.

Why not microservices: a single-user-facing MVP with ~20 domain concepts does not justify service
boundaries, distributed tracing complexity, or multi-repo release management. The financial engine
is pure Python and trivially extractable later if a documented need appears.

```
┌──────────────────────────────────────────────┐
│  Next.js (App Router, TS)                    │
│  server actions / typed fetch to /v1         │
└──────────────────┬───────────────────────────┘
                   │ HTTPS + JWT (httpOnly cookie)
┌──────────────────▼───────────────────────────┐
│  FastAPI application (single container)      │
│                                              │
│  api/         routers, deps, DTOs, errors    │
│  services/    use cases, transactions        │
│  domain/      PURE: money, financial engine  │
│  ai/          gateway, prompts, guardrails   │
│  infra/       db, repos, config, telemetry   │
└───────┬───────────────────────┬──────────────┘
        │ SQLAlchemy async      │ LiteLLM
┌───────▼──────────┐   ┌────────▼─────────────┐
│  PostgreSQL      │   │  LLM providers       │
│  (+ Redis later) │   │  (OpenAI/Groq/...)   │
└──────────────────┘   └──────────────────────┘
        │
        ├── OpenTelemetry → collector → backend
        └── Langfuse (LLM spans, redacted)
```

## 2. Component responsibilities

| Layer | Owns | Must never |
| --- | --- | --- |
| `api/` | HTTP contract, auth dependency, validation, status codes, error mapping | Contain business rules or SQL |
| `services/` | Use cases, transaction boundaries, authorization checks, orchestration of engine + persistence | Compute financial values inline |
| `domain/` | Money type, formulas, comparison matrices, goal/emergency logic | Import FastAPI, SQLAlchemy, HTTP clients, or LLM SDKs |
| `ai/` | ModelGateway protocol, LiteLLM adapter, prompt templates, grounding validator, output guardrails, context builder | Mutate domain data or execute tools that write |
| `infra/` | DB session, repositories, config, secrets loading, telemetry, seed data, jobs | Leak provider SDK types upward |

Dependency direction is one-way: `api → services → domain`, `services → infra`, `ai → domain`
(types only). Enforced by a CI import-graph test.

## 3. Request flows

### 3.1 Deterministic compute (no AI)

```
Client → GET /v1/metrics/savings
  → auth dependency (JWT → user + org)
  → service loads profile + expenses (org-scoped, parameterised)
  → domain.compute_metrics(profile) → frozen dataclass w/ provenance
  → response envelope + trace_id
```

No AI in the path. Latency target < 100 ms.

### 3.2 Comparison / scenario

```
Client → POST /v1/comparisons {instrument_ids, horizon, contribution}
  → validate comparability
  → resolve rates (verified + fresh) else 422 missing_input
  → domain.compare(...) → matrix + assumptions + provenance
  → persist computed_result (id, org, payload, expires_at)
  → 200 {result, result_ref}
```

### 3.3 AI explanation

```
Client → POST /v1/insights/explain {result_ref}
  → auth + org check on result_ref (404 if foreign)
  → scope/intent gate (rules first; model-assisted classification optional)
  → context builder: result payload only (+ approved memory prefs)
  → prompt template (versioned) → ModelGateway.complete()
  → schema validation → grounding check (numbers ⊆ payload)
  → forbidden-language scan → disclaimer injection
  → trace to Langfuse (redacted) → 200
```

The client never supplies numbers to the AI path. `result_ref` prevents numeric injection (ADR-0009).

### 3.4 Memory

```
User action "remember this preference" → POST /v1/memory
  → validator (no secrets, no account numbers, length cap)
  → row: scope=user, provenance=explicit_user_confirmation
Reads: context builder queries by (org_id, user_id, kind) with limits.
```

## 4. Cross-cutting concerns

| Concern | Approach |
| --- | --- |
| Auth | JWT access (15 min) + rotating refresh token family; Argon2id |
| Tenancy | `org_id` on every tenant table; repository filters; token-derived org only |
| Errors | Stable machine codes (`email_taken`, `missing_input`, `rate_limited`), no stack traces in bodies |
| Money | `Decimal` end-to-end, `Numeric(18,2)` in DB, 2 dp at presentation boundary only |
| Time | UTC storage, `Asia/Kolkata` presentation, injected clock in `domain/` |
| Config | Pydantic Settings; fail-fast validation at startup |
| Background | One in-process scheduler job (account purge, result TTL cleanup) — no Celery in MVP |
| Feature flags | Env-based booleans only |

## 5. Data stores

| Store | Used for | MVP |
| --- | --- | --- |
| PostgreSQL | All durable data | Required |
| Redis/Valkey | Rate-limit counters, caching of rate lookups, idempotency keys | Optional; in-process fallback for single-instance MVP |
| pgvector | Semantic memory | **Not used** (ADR-0007) |
| Object storage | Document uploads | **Not used** |

## 6. Deployment view

- Frontend: Next.js on a static/serverless host (Vercel).
- Backend: single container on a container host (Fly.io / Railway / Render) behind HTTPS.
- Database: managed PostgreSQL (Neon / Supabase / RDS-compatible), private networking where possible.
- Telemetry: OTel collector sidecar or direct OTLP export; Langfuse cloud (redacted).

## 7. Scalability statement

The MVP targets < 1,000 users and < 50 req/s. Vertical scaling on one container plus connection
pooling (pgbouncer or SQLAlchemy pool with `pool_size=10`) is sufficient. Horizontal scaling is
possible if rate-limit state moves to Redis; the app holds no other local state.

## 8. Explicit architectural non-goals

- No microservices, message broker, or event sourcing.
- No multi-agent graph, planner loop, or autonomous tool use.
- No CQRS/read-model layer; Postgres + indexes is enough.
- No GraphQL; a typed REST/OpenAPI surface is sufficient for one client.
- No plugin system, no multi-currency abstraction, no offline mode.

## 9. Architecture risks

| Risk | Mitigation |
| --- | --- |
| Layer erosion (engine logic creeping into API) | Import-graph CI test + review checklist |
| Rate data drift making comparisons wrong | Versioned rates + staleness flags + refusal to compute (FR-RATE-03) |
| AI provider capability assumptions | Capability registry validated at startup (NFR-AI-03) |
| Single-instance limits (in-process rate limiting) | Documented limit; Redis is a drop-in swap |
| PII in traces | Redaction middleware + span-attribute regex test (AC-OBS-03) |
