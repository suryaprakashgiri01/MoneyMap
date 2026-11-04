# ADR-0010 — Org-Scoped Tenancy with Personal Orgs at MVP

**Status:** Accepted · **Date:** 2026-11-04 · **Stage:** 00

## Context

The specification defines three memory scopes including organization memory, and requires strict user
data isolation. MVP serves individuals, but a later shared-workspace feature (couples, families, small
teams) is plausible. Retrofitting tenancy into a user-only schema is painful and error-prone.

## Decision

1. Every tenant-scoped table carries `org_id` from day one.
2. Signup creates exactly one `orgs` row ("Personal workspace") and links the user to it.
3. `org_id` is derived **from the token**, never from a request body or path.
4. Repositories require an explicit `org_id` argument (no implicit global queries); a missing filter
   is a type error, not a runtime surprise.
5. Cross-org access returns **404**, not 403, to avoid resource enumeration. The attempt is logged to
   `security_events`.
6. Shared workspaces need a `memberships` table and role checks — deferred, not designed away.

## Consequences

- All queries carry one extra predicate; index every tenant table on `(org_id, user_id, …)`.
- Isolation is testable in one place and asserted for every resource (AC-TEN-01..03).
- A one-user org is a harmless no-op today and free optionality later.

## Alternatives considered

- **User-only schema, add orgs later** — rejected: migration churn across every table plus a security
  rewrite at the worst time.
- **Row-level security in Postgres** — deferred: attractive defence-in-depth, but requires session
  variable discipline and complicates migrations; revisit in Stage 09 hardening.

## Compliance

- FR-TEN-01..03, NFR-PRIV-01..07, AC-TEN-01..03.
