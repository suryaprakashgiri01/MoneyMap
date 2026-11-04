# Stage 07 — Observability

**Branch:** `stage/07-observability`
**Spec:** FR-OBS-01..03, NFR-OBS-01..04, NFR-PRIV-01..03, ADR-0013, AC-OBS-01..03

## Objective

Trace every request and AI call without exporting user money or identity.

## Tasks

| ID | Task | Notes |
| --- | --- | --- |
| T07-01 | OTel SDK setup | resource attrs, exporter from env |
| T07-02 | HTTP + DB instrumentation | one trace per request |
| T07-03 | Correlation | `request_id` ↔ `trace_id`, response header |
| T07-04 | AI spans | provider, model, prompt_version, tokens, retries, grounding outcome |
| T07-05 | Langfuse integration | metadata only, redacted |
| T07-06 | `Redactor` + attribute allow-list | pseudonymous actor hash |
| T07-07 | Log schema | JSON, no bodies, field names only on errors |
| T07-08 | Metrics | rate, errors, p95, retry count, grounding failures, cost |
| T07-09 | Alerts | 5xx, AI unavailable, readiness, grounding failure |
| T07-10 | PII-leak CI test | regex over exported spans/logs |
| T07-11 | Dashboard definitions | JSON/markdown documented |

## Exit criteria

- AC-OBS-01..03 pass; no amounts or emails in any exported signal.
- One trace per request, AI spans fully attributed.
- Alert rules fire in a staging drill.

## Out of scope

SLO management, on-call rotation, long-term log retention.

## Risks

| Risk | Mitigation |
| --- | --- |
| SDK default attributes leaking request bodies | allow-list (deny-by-default) |
| Cost of 100% AI tracing | sampling policy + short retention |
| Langfuse outage affecting requests | tracing is best-effort, never blocking |
