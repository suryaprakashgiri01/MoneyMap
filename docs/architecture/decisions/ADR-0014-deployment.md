# ADR-0014 — Deployment Topology: Single Container + Managed Postgres + Vercel

**Status:** Accepted · **Date:** 2026-11-04 · **Stage:** 00

## Context

Requirement S15: a publicly accessible MVP. Constraints: one engineer, low budget, must run locally
identically, must not add ops burden that outweighs the product.

## Decision

1. **Local**: `docker compose up` → `api` (FastAPI/uvicorn), `postgres:16`, optional `redis`, `otel-collector`.
   Frontend runs via `npm run dev` (or a compose profile).
2. **Production**:
   - Backend: one container image (multi-stage, non-root user, `uv`-installed deps, healthcheck) on
     Fly.io (primary choice) or Railway/Render.
   - Database: managed PostgreSQL (Neon) with TLS; `DATABASE_URL` injected as a secret.
   - Frontend: Next.js on Vercel, `NEXT_PUBLIC_API_BASE_URL` per environment.
   - Traces: OTLP export to a hosted backend; Langfuse cloud, redacted.
3. **Migrations**: run as a release step (`alembic upgrade head`) before the new app version serves
   traffic; the app refuses to start on schema mismatch (`/readyz` fails).
4. **Secrets**: platform secret manager only; never in images, never in compose files committed to git.
5. **Environments**: `local`, `staging` (optional, same image), `production`. Config via env vars with
   Pydantic validation at startup.
6. **Backups**: managed daily snapshots; restore procedure documented in the Stage 09 runbook.

## Consequences

- Zero Kubernetes, zero Terraform in MVP (documented as future work).
- Cold-start and single-instance limits: in-process rate limiting is acceptable until Redis is enabled.
- Vendor choice is reversible: the image is portable; only secrets and hostnames change.

## Alternatives considered

- **Single VPS with docker compose** — viable fallback; weaker managed backups and TLS handling.
- **Kubernetes** — rejected: unjustified operational load for < 1,000 users.
- **AWS Lambda + API Gateway** — rejected: cold starts and local-parity cost.
- **Vercel-only full-stack** — rejected: Python backend and long-lived DB connections fit poorly.

## Compliance

- AC-DEP-01..04, NFR-COST-02/03, NFR-REL-01/04.
