# Stage 01 — Backend Foundation

**Branch:** `stage/01-backend-foundation`
**Spec:** FR-AUTH-01..05, FR-TEN-01..03, NFR-SEC-*, NFR-REL-01/04, ADR-0010, ADR-0012, ADR-0014

## Objective

A runnable, testable API skeleton with real authentication, tenant-scoped persistence, migrations and
CI — no financial logic yet.

## Tasks

| ID | Task | Notes |
| --- | --- | --- |
| T01-01 | Project scaffolding | `uv` + `pyproject.toml` (pinned Python 3.12/3.13), `ruff`, `mypy --strict`, layout `api/services/domain/ai/infra` |
| T01-02 | Settings + config validation | Pydantic Settings, fail-fast, `.env.example` |
| T01-03 | Docker Compose | api + postgres (+ optional redis), healthchecks |
| T01-04 | SQLAlchemy async engine + session | `asyncpg`, pool config, dependency injection |
| T01-05 | Base models + `orgs`/`users` | UUID PKs, timestamps, soft delete, `org_id` |
| T01-06 | Alembic baseline migration | 1 revision, verified on a clean DB |
| T01-07 | Password hashing (Argon2id) | tuned params, constant-time verify |
| T01-08 | Signup + login | personal org creation, generic errors |
| T01-09 | JWT access + rotating refresh | hash storage, family reuse detection, logout-all |
| T01-10 | Auth dependencies + error codes | `/v1` protection, stable codes |
| T01-11 | Health endpoints | `/healthz`, `/readyz` (DB + schema revision) |
| T01-12 | Rate limiting | in-process limiter, Redis-ready interface |
| T01-13 | Structured logging + request id | JSON logs, no bodies |
| T01-14 | CI pipeline | lint → unit → integration (PG service) → coverage |
| T01-15 | Import-graph test | enforce layering |
| T01-16 | Auth + isolation tests | AC-AUTH-*, AC-TEN-01 baseline |

## Exit criteria

- All AC-AUTH-* pass; token reuse revokes the family.
- `docker compose up && make test` works from a clean clone.
- Migrations apply cleanly and `/readyz` fails on a schema mismatch.
- No secrets or PII in logs; identity check green.

## Out of scope

Financial tables, AI, memory, frontend.

## Risks

| Risk | Mitigation |
| --- | --- |
| Wheel availability for 3.12 vs local 3.15 | pin with `uv`; verify `asyncpg`, `argon2-cffi` build on target |
| Refresh-token edge cases | explicit tests for rotation, expiry, reuse, logout-all |
