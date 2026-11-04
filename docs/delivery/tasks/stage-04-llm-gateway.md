# Stage 04 — LLM Gateway & Explanation Pipeline

**Branch:** `stage/04-llm-gateway`
**Spec:** FR-AI-01..06, NFR-AI-01..05, ADR-0004, ADR-0005, ADR-0009, SPEC-ARCH-003

## Objective

A provider-independent model layer and the grounded "explain this result" pipeline, testable offline.

## Tasks

| ID | Task | Notes |
| --- | --- | --- |
| T04-01 | `ModelGateway` protocol + DTOs | `CompletionRequest`, `CompletionResult`, capabilities |
| T04-02 | LiteLLM adapter | thin, error mapping, no SDK leakage |
| T04-03 | Capability registry (YAML) | validated at startup |
| T04-04 | Error taxonomy + retry policy | transient only, bounded backoff |
| T04-05 | `FakeGateway` for tests | scripted responses, offline suite |
| T04-06 | Prompt template loader + versioning | front-matter, hash, `prompt_version` |
| T04-07 | Template `explain.metrics.v1` | payload pruning, verbatim-number instruction |
| T04-08 | Payload pruner per result kind | field allow-list |
| T04-09 | Grounding validator | parsing, normalisation, allowed-set check |
| T04-10 | `POST /v1/insights/explain` | `result_ref` ownership + TTL |
| T04-11 | Schema validation + one repair retry | then fail closed |
| T04-12 | `status` semantics | ok/unavailable/incomplete/refused |
| T04-13 | Config-only provider switch test | two fake adapters |
| T04-14 | Cost/latency guards | max_tokens, timeout, request budget |

## Exit criteria

- AC-AI-01..06 pass with mocked providers.
- Grounding rejects an invented number and accepts every payload number (table-driven).
- Provider timeout returns the engine result with `explanation_status: unavailable`.
- No provider string appears outside `ai/adapters/`.

## Out of scope

Memory, guardrail suite, production tracing, additional templates.

## Risks

| Risk | Mitigation |
| --- | --- |
| Grounding false negatives (valid paraphrase rejected) | normalisation rules + verbatim-quote instruction; tracked as eval failure |
| LiteLLM abstraction leaks | adapter ≤ 150 lines, error mapping tests |
| Structured-output variance across providers | capability-driven mode + post-parse validation |
