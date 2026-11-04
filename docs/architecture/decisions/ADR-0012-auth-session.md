# ADR-0012 — JWT Access + Rotating Refresh Tokens, Argon2id

**Status:** Accepted · **Date:** 2026-11-04 · **Stage:** 00

## Context

The app stores financial data and must be safe on shared devices. Options: server-side sessions
(DB/Redis), stateless JWT, or a hybrid. Requirements: short-lived credentials, revocable sessions,
"log out all devices", safe storage in a Next.js frontend.

## Decision

1. **Access token**: JWT, HS256 (secret from config, `kid` header for rotation), 15-minute TTL,
   claims `sub`, `org`, `jti`, `iat`, `exp`, `ver`.
2. **Refresh token**: opaque 256-bit random value, stored **hashed** (SHA-256) in `refresh_tokens`,
   30-day TTL, rotated on every use, grouped by `family_id`.
3. **Reuse detection**: presenting an already-rotated refresh token revokes the entire family and
   writes a `security_events` row.
4. **Storage**: refresh token in `httpOnly`, `Secure`, `SameSite=Lax` cookie scoped to
   `/v1/auth`; access token kept in memory on the client (not localStorage).
5. **Passwords**: Argon2id via `argon2-cffi` with tuned parameters (≥ 19 MiB, t=2, p=1), constant-time
   verification, generic error messages.
6. **CSRF**: refresh endpoint requires `SameSite=Lax` + origin check; state-changing endpoints use the
   bearer access token, not cookies.
7. **Logout all devices**: revoke every family for the user.

## Consequences

- Stateless request auth (no DB hit per request) with real revocation via short TTL.
- One extra round trip on access-token expiry; the client handles 401 `token_expired` with a single
  refresh retry.
- Redis is not required for MVP; refresh state is in Postgres.

## Alternatives considered

- **Server-side sessions in Redis** — viable, but adds a required dependency for MVP and a second
  store to operate; tokens also serve future API clients.
- **Long-lived JWT** — rejected: no revocation.
- **Cookie-only sessions with CSRF tokens** — rejected for API ergonomics; the frontend is not the
  only intended consumer.

## Compliance

- FR-AUTH-01..05, NFR-SEC-01/02/08, AC-AUTH-01..06.
