# Stage 08 — Frontend & Deployment

**Branch:** `stage/08-frontend-deployment`
**Spec:** S1–S15, NFR-UX-01..05, ADR-0012, ADR-0014, AC-DEP-01..04

## Objective

A usable, accessible Next.js app and a public HTTPS deployment.

## Tasks

| ID | Task | Notes |
| --- | --- | --- |
| T08-01 | Next.js scaffold + typed API client | App Router, TS strict, OpenAPI types |
| T08-02 | Auth flow | signup/login/refresh/logout-all, refresh in httpOnly cookie |
| T08-03 | Profile + expenses forms | Indian number grouping, inline help |
| T08-04 | Dashboard metrics | surplus, savings ratio, expense ratio |
| T08-05 | Emergency fund + goals views | gap, coverage, feasibility |
| T08-06 | Holdings manager | type-specific forms incl. LIC |
| T08-07 | Comparison table | provenance badges, `partial`/`stale` states, card fallback |
| T08-08 | Scenario builder + diff | baseline vs scenario |
| T08-09 | Explanation panel | grounding badge, disclaimer, feedback control |
| T08-10 | Memory consent UI | exact text preview, list, delete |
| T08-11 | Accessibility pass | keyboard, focus, contrast, labels, WCAG 2.1 AA |
| T08-12 | Error/empty/loading states | designed, not defaulted |
| T08-13 | Container image + Fly config | non-root, healthcheck, release migration |
| T08-14 | Managed Postgres + secrets | TLS, least privilege |
| T08-15 | Deploy + smoke test | AC-DEP-01..04 |
| T08-16 | Privacy notice + disclaimers live | DISC-01..05 |

## Exit criteria

- AC-DEP-01..04 pass; public URL live over HTTPS.
- Lighthouse accessibility ≥ 90 on dashboard and comparison pages.
- No token in `localStorage`; no third-party analytics on authenticated pages.
- Every rendered number shows provenance and units.

## Out of scope

Native apps, i18n beyond `en-IN`, dark mode, offline mode, shared workspaces UI.

## Risks

| Risk | Mitigation |
| --- | --- |
| Frontend re-implementing calculations | all numbers come from API; no client arithmetic |
| Wide comparison tables on mobile | card fallback at < 768 px |
| Deploy drift from local | same image, config-only differences |
