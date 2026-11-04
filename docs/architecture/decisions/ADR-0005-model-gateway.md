# ADR-0005 — ModelGateway Abstraction with LiteLLM Adapter

**Status:** Accepted · **Date:** 2026-11-04 · **Stage:** 00

## Context

The product must not be coupled to one provider. Providers differ in structured-output support, tool
calling, streaming, context limits, error semantics, and retention policy. Requirements NFR-AI-01..05
mandate a gateway and capability metadata.

## Decision

1. Define a `ModelGateway` protocol in `ai/gateway.py` with `complete`, optional `stream`, and
   `capabilities`.
2. Implement a single adapter `ai/adapters/litellm_adapter.py` using LiteLLM as the transport for all
   providers. No provider SDK is imported outside `ai/adapters/`.
3. Model selection is configuration: `AI_PROVIDER`, `AI_MODEL`, `AI_FALLBACK_MODEL`, `AI_TIMEOUT_S`,
   `AI_MAX_TOKENS`, `AI_TEMPERATURE`.
4. Capabilities are declared in `infra/config/models.yaml` and validated at startup against the
   selected model's requirements (e.g. `requires_structured_output: true`).
5. Errors are normalised into a small taxonomy (`RateLimited`, `Timeout`, `AuthFailed`,
   `InvalidRequest`, `ProviderError`, `ContentFiltered`); retries only on transient classes.
6. Business code depends on the protocol; tests use a fake gateway (`FakeGateway`) with scripted
  responses, so the suite runs offline.

## Consequences

- Provider switching is a config change; documented per-provider limitations live in SPEC-ARCH-003 §8.
- LiteLLM is a heavy dependency and its own abstraction can leak; mitigated by keeping the adapter
  thin (< 150 lines) and mapping errors explicitly.
- Structured output support varies: the gateway exposes a `structured_output_mode`
  (`json_schema` | `json_object` | `prompt_only`) per model.

## Alternatives considered

- **Direct SDKs per provider** — rejected: N providers × M features of maintenance.
- **LangChain `ChatModel` as the abstraction** — rejected: framework lock-in and a much larger
  dependency surface than needed.
- **Own HTTP client per provider** — rejected at MVP; revisit only if LiteLLM blocks a needed feature.

## Compliance

- AC-AI-05 (config-only switch), AC-AI-06 (capability validation at startup), NFR-AI-01..04.
