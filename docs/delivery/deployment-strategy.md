# Deployment Strategy

**Document ID:** SPEC-DLV-001 · **Stage:** 00 · **Status:** Approved v1.0
**Related:** ADR-0014, NFR-REL-01..04, AC-DEP-01..04

## 1. Environments

| Env | Purpose | Backend | DB | Frontend |
| --- | --- | --- | --- | --- |
| local | development + tests | `docker compose` (uvicorn --reload) | `postgres:16` container | `npm run dev` |
| staging (optional) | pre-release verification | same image, small instance | managed PG (separate db) | Vercel preview |
| production | public MVP | single container (Fly.io) | managed PG (Neon) | Vercel |

Config is environment-only: no code branches on environment name.

## 2. Local development

```
docker compose up -d          # api + postgres + (optional) redis, otel-collector
uv run alembic upgrade head
uv run python -m app.infra.seed  # rate dataset + generic memory + synthetic demo user
uv run pytest
```

- Compose file is committed; `.env.example` lists every required variable (committed, no values).
- Frontend: `npm ci && npm run dev` with `NEXT_PUBLIC_API_BASE_URL=http://localhost:8000`.
- One command (`make dev`) brings both up.

## 3. Container image

- Multi-stage: `python:3.12-slim` builder with `uv sync --frozen` → runtime stage without build tools.
- Non-root user (`app`), read-only root filesystem where possible, healthcheck on `/healthz`.
- Pinned base image digest; rebuilt weekly and on security advisories.
- `PYTHONUNBUFFERED=1`, sane `uvicorn` workers (1 process, async concurrency; scale vertically first).

## 4. Release process

1. PR merged to `main` after stage gate.
2. CI builds and tags image: `app:<git-sha>` + `app:latest`.
3. Deploy step runs `alembic upgrade head` as a release command (single instance ⇒ no concurrency).
4. New version starts; `/readyz` must pass (DB reachable, schema revision, required config present).
5. Smoke test: signup, login, metrics, explain (mocked-off AI acceptable in staging).
6. If `/readyz` fails twice ⇒ rollback to previous image tag (migrations are backward compatible for one
   release; see §6).

## 5. Configuration

| Variable | Required | Notes |
| --- | --- | --- |
| `ENV` | yes | `local` \| `staging` \| `production` |
| `DATABASE_URL` | yes | secret; `postgresql+asyncpg://…?sslmode=require` |
| `JWT_SECRET`, `JWT_KID` | yes | secret; rotation supported |
| `REFRESH_PEPPER` | yes | secret; hashes refresh tokens |
| `AI_PROVIDER`, `AI_MODEL` | yes for AI | e.g. `openrouter` + model id |
| `AI_API_KEY` | yes for AI | secret |
| `AI_FALLBACK_MODEL` | no | optional |
| `AI_TIMEOUT_S`, `AI_MAX_TOKENS`, `AI_TEMPERATURE` | no | defaults in settings |
| `RATE_STALENESS_DAYS__*` | no | per-product staleness windows |
| `CORS_ORIGINS` | yes | comma-separated exact origins |
| `OTEL_EXPORTER_OTLP_ENDPOINT` | no | unset ⇒ spans dropped in local |
| `LANGFUSE_PUBLIC_KEY`, `LANGFUSE_SECRET_KEY` | no | unset ⇒ AI tracing disabled |
| `REDIS_URL` | no | unset ⇒ in-process rate limiter |

Validation is fail-fast: a missing required value aborts startup with a clear message (no secrets in logs).

## 6. Database & migrations

- Alembic forward-only. Each migration must be compatible with the previous app version for one release
  (add nullable columns, backfill, then tighten in a later release).
- No destructive migration in the same release as code that stops using the column.
- Snapshots: managed daily; retention 7 days in MVP.
- Restore drill: documented command sequence, executed once in Stage 09.

## 7. Frontend

- Next.js App Router; API URL from `NEXT_PUBLIC_API_BASE_URL`.
- Auth: access token in memory + refresh cookie (see ADR-0012); no token in `localStorage`.
- No third-party analytics on authenticated pages.
- Security headers (CSP, HSTS, X-Content-Type-Options) set in `next.config` / host config.

## 8. Observability in production

- OTLP export with 10% sampling for non-AI, 100% for AI and errors.
- Alerts (minimal MVP): 5xx rate > 1% (5 min), AI `unavailable` rate > 10% (15 min),
  `/readyz` failing, grounding-failure count > 0 in 1 h.
- Dashboards: request rate/latency/errors, AI latency/cost/retry, engine error count.

## 9. Backups, DR, and limits

| Item | MVP approach |
| --- | --- |
| RPO | 24 h (daily snapshot) |
| RTO | 4 h (redeploy + restore) |
| Multi-region | No |
| Availability target | 99% best effort |
| Comms on outage | Status note in README/social; no SLA |

## 10. Cost envelope (MVP)

| Item | Estimate |
| --- | --- |
| App instance | USD 5–10 / month |
| Managed Postgres | USD 0–10 / month (free/low tier) |
| Frontend host | USD 0 (hobby) |
| LLM usage | < USD 5 / month at MVP volume (< 500 explanations/day) |
| Observability | USD 0–20 / month |

## 11. Launch checklist

- [ ] Secrets provisioned in platform, not in repo.
- [ ] `alembic upgrade head` runs clean on a fresh database.
- [ ] Seed dataset loaded; `verified_by` populated.
- [ ] HTTPS + HSTS verified; CORS limited to the deployed origin.
- [ ] Rate limits active; AI daily cap configured.
- [ ] Disclaimers visible; privacy notice linked.
- [ ] `/healthz` and `/readyz` monitored.
- [ ] Rollback rehearsed.
