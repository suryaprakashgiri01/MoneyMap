# ADR-0013 — Observability with Privacy-Safe Telemetry (OTel + Langfuse)

**Status:** Accepted · **Date:** 2026-11-04 · **Stage:** 00

## Context

We must debug AI failures and latency without exporting users' financial data. Observability tools are
data-export channels and therefore a privacy risk surface (NFR-PRIV-01..03, NFR-OBS-01..04).

## Decision

1. **OpenTelemetry** for traces/metrics/logs, exported via OTLP to a collector (or directly to a backend
   in MVP). One trace per HTTP request; spans for DB, engine calls, and AI calls.
2. **Langfuse** for LLM observability: prompt version, model, provider, latency, token usage, retry
   count, grounding outcome, eval flags.
3. **Redaction by construction**: a `Redactor` wraps every exporter. It:
   - replaces user identifiers with `sha256(id + server_pepper)[:12]`,
   - masks/drops any attribute matching amount patterns (`₹`, `INR`, 4+ digit runs),
   - keeps only allow-listed attribute keys (no free-form `metadata` pass-through).
4. **Logging**: structured JSON, `request_id`/`trace_id`/route/status/duration only. No bodies, no
   prompts, no amounts. Validation errors log field names, not values.
5. **Sampling**: 100% of errors, 100% of AI spans, 10% of successful non-AI requests.
6. **No session replay, no third-party analytics on authenticated pages.**
7. A CI test asserts exported span attributes never match currency/number patterns (AC-OBS-03).

## Consequences

- Some debugging requires reproducing locally with synthetic data — an intentional cost.
- Prompt text is not visible in Langfuse; prompt *version* is. Template diffs live in git.
- Cost: a small redaction layer on every export path.

## Alternatives considered

- **Full prompt logging for convenience** — rejected: prompt contains amounts.
- **No LLM observability** — rejected: provider regressions and grounding failures become invisible.
- **Self-hosted Langfuse at MVP** — deferred: operational cost without benefit at this scale.

## Compliance

- FR-OBS-01..03, NFR-OBS-01..04, AC-OBS-01..03, AC-PRIV-01/03.
