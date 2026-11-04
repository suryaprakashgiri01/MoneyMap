# AI Workflow Specification

**Document ID:** SPEC-ARCH-003 · **Stage:** 00 · **Status:** Approved v1.0
**Related:** ADR-0004 (deterministic first), ADR-0005 (gateway), ADR-0006 (no agents), ADR-0009 (explanation scope)

## 1. Principle

The LLM is a **narrator**, not a calculator. Its only job is to turn a server-computed result into
plain language that is verifiable against that result. Every step before the model call is
deterministic; every step after it is a validator.

## 2. Endpoint contract

`POST /v1/insights/explain`

```jsonc
// request
{ "result_ref": "b6f1…", "style": "concise" | "detailed", "locale": "en-IN" }

// response 200
{
  "explanation": "Based on the inputs you provided, …",
  "status": "ok" | "unavailable" | "incomplete" | "refused",
  "grounding": { "checked": true, "unmatched_numbers": 0, "payload_hash": "sha256:…" },
  "model": { "provider": "openrouter", "id": "…", "prompt_version": "explain.metrics.v1" },
  "disclaimer": "This is an educational analysis …",
  "trace_id": "…",
  "request_id": "…"
}
```

Notes:
- The client cannot pass amounts. Only a `result_ref` owned by the caller's org is accepted.
- `result_ref` must be unexpired (24 h) and of kind compatible with the prompt template.

## 3. Pipeline (10 steps)

| # | Step | Deterministic? | Failure behaviour |
| --- | --- | --- | --- |
| 1 | AuthN/AuthZ + org scoping of `result_ref` | yes | 401/404, no model call |
| 2 | Request validation (Pydantic strict) | yes | 422 |
| 3 | Scope gate: rule-based classifier for forbidden intents (transaction, guarantee, tax filing, market timing, "which is best") | yes (rules) | `status: refused` + canonical text, no model call |
| 4 | Load result payload from `computed_results` | yes | 404 `result_expired` |
| 5 | Context assembly: payload + ≤3 approved user preferences + glossary terms | yes | degrade to payload-only |
| 6 | Prompt construction from versioned template + capability-aware params (temperature 0.2, max_tokens cap) | yes | 500 config error |
| 7 | `ModelGateway.complete()` with timeout, bounded retries | no | `status: unavailable` + 200 + engine result still usable |
| 8 | Schema validation (JSON/structured output or post-parse) | yes | retry once with repair prompt, then `unavailable` |
| 9 | Grounding check: extract every numeric token; each must appear in payload (with unit/scale normalisation) | yes | retry once at temperature 0, then `unavailable` (fail closed) |
| 10 | Output guardrails: forbidden patterns, disclaimer injection, length cap, redaction of anything unexpected; emit trace | yes | `status: refused` if language violates policy |

Steps 3 and 9 are the two places where the product either stays honest or does not ship.

## 4. Grounding algorithm (step 9)

1. Tokenise the explanation; extract candidate numbers with Indian and Western grouping
   (`1,00,000` and `100,000`), percentages, and currency prefixes.
2. Normalise: strip separators, parse `Decimal`, map `lakh`/`crore` suffixes, align percent vs
   fraction (`7.1%` ⇒ `0.071`).
3. Build the allowed set from the payload's numeric leaves **plus** derived display values the engine
   explicitly exported (`display` strings are part of the payload for this reason).
4. Any number not in the allowed set → grounding failure. Allow-listed non-financial integers
   (years, month counts, counts of items) come from `payload.derived.counts`.
5. Record `unmatched_numbers` and the offending tokens in the trace (tokens only, no amounts).

Deliberate trade-off: a strict matcher occasionally rejects a legitimate paraphrase
("about 8.5 lakh" when the payload says `₹8,50,000`). Mitigation is normalisation rules plus
prompting the model to quote payload numbers verbatim; false rejections are logged as eval failures.

## 5. Prompt versioning

- Templates live in `backend/app/ai/prompts/<name>.<version>.md` with a YAML front-matter block
  (`version`, `model_capabilities_required`, `max_input_tokens`, `schema`).
- `prompt_version` is recorded on every response, trace, and feedback row.
- Changing a template requires: eval run, diff review, and a `docs/` note in the stage PR.

## 6. Template inventory (MVP)

| Template | Input | Notes |
| --- | --- | --- |
| `explain.metrics.v1` | savings metrics payload | never restates advice |
| `explain.emergency_fund.v1` | EF target/gap/coverage | gap language must be neutral |
| `explain.goal.v1` | goal progress + feasibility | says "shortfall", not "you must save more" |
| `explain.lic_analysis.v1` | XIRR + comparison rows | no directive verbs |
| `explain.comparison.v1` | metric matrix | no ranking language |
| `explain.scenario.v1` | baseline vs scenario deltas | states assumptions explicitly |
| `refusal.v1` | reason code | static text, no model call needed |

`refusal.v1` is a static string table, not a generation task, so out-of-scope requests never reach a
provider.

## 7. ModelGateway interface

```python
class ModelGateway(Protocol):
    async def complete(self, req: CompletionRequest) -> CompletionResult: ...
    async def stream(self, req: CompletionRequest) -> AsyncIterator[CompletionChunk]: ...  # optional
    def capabilities(self, model_id: str) -> ModelCapabilities: ...
```

```python
@dataclass(frozen=True)
class CompletionRequest:
    messages: tuple[Message, ...]
    model: str
    temperature: Decimal
    max_tokens: int
    response_schema: type[BaseModel] | None
    timeout_s: float
    metadata: Mapping[str, str]   # prompt_version, result_kind, request_id

@dataclass(frozen=True)
class ModelCapabilities:
    structured_output: bool
    tool_calling: bool
    streaming: bool
    context_window: int
    max_output_tokens: int
    embeddings: bool
```

Normalised error taxonomy: `RateLimited`, `Timeout`, `AuthFailed`, `InvalidRequest`, `ProviderError`,
`ContentFiltered`. Retry only on `RateLimited`, `Timeout`, `ProviderError` (max 2 attempts,
exponential backoff with jitter, total ≤ timeout budget).

## 8. Provider capability matrix (initial, config-driven)

| Provider | Structured output | Tools | Streaming | Notes |
| --- | --- | --- | --- | --- |
| OpenAI-compatible | yes (json_schema) | yes | yes | reference implementation |
| Groq | partial (json_object) | partial | yes | validate schema post-parse |
| Gemini | yes | yes | yes | message-role mapping differs |
| OpenRouter | model-dependent | model-dependent | yes | capability per underlying model |
| Cerebras | partial | no | yes | no tool calling |
| Local (Ollama/vLLM) | partial | no | yes | no external retention |

Rules:
- The matrix is data (YAML), not code branches.
- Startup validation fails if the selected model is required to support structured output but the
  registry says it does not. Fallback: post-parse validation path, explicitly configured.
- No provider-specific strings outside `ai/adapters/`.

## 9. Guardrails around the model

| Guardrail | Implementation |
| --- | --- |
| Injection | user/retrieved text is fenced as data with a fixed instruction hierarchy; `notes` fields never concatenated into system prompt |
| Tool permissions | tool set is empty in MVP-0; if added, read-only allow-list + `NFR-SEC` review |
| Output policy | forbidden-pattern scan (`non-goals-and-disclaimers.md` §4) before return |
| Length | `max_tokens` cap + post-check |
| PII | output scanned for email/phone/account-number patterns; abort if found |
| Rate limit | per-user AI budget (10 req/min, daily cap) |

## 10. Failure semantics

| Situation | HTTP | `status` | User sees |
| --- | --- | --- | --- |
| Provider down/timeout | 200 | `unavailable` | numbers + "explanation unavailable, try again" |
| Grounding fails twice | 200 | `unavailable` | numbers only; trace flagged |
| Result incomplete | 200 | `incomplete` | fixed text naming missing field |
| Out of scope | 200 | `refused` | canonical refusal |
| Invalid `result_ref` | 404 | — | error page |
| No AI config | 200 | `unavailable` | numbers only |

The deterministic result is **always** returned; AI failure never removes user value
(NFR-REL-02).

## 11. Cost & latency budget

- Input ≤ 1,200 tokens (payload pruned to the fields the template needs).
- Output ≤ 400 tokens.
- One retry only; 20 s hard timeout.
- Target: p95 < 8 s, < USD 0.01 per explanation at default config.

## 12. Explicitly not built

- Multi-agent planning or self-critique loops.
- Autonomous tool execution against the database.
- Conversation memory as a chat transcript.
- Streaming in MVP (single complete response keeps validation simple).
- Fine-tuning or distillation on user data.
