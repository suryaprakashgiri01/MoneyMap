# Stage Roadmap

**Document ID:** SPEC-DLV-002 · **Stage:** 00 · **Status:** Approved v1.0

Ten stages, each on its own branch, each with a task file, each gated by acceptance criteria.
Nothing from a later stage is implemented early "because it is easier".

| Stage | Branch | Outcome | Key specs |
| --- | --- | --- | --- |
| 00 | `stage/00-product-spec` | Approved spec baseline + ADRs | all `docs/` |
| 01 | `stage/01-backend-foundation` | Runnable API, auth, DB, migrations, CI | FR-AUTH-*, NFR-SEC-*, ADR-0010/0012 |
| 02 | `stage/02-financial-engine` | Pure deterministic engine, fully tested | FR-CALC-*, NFR-COR-*, ADR-0003/0004 |
| 03 | `stage/03-user-financial-profile` | Profile, metrics, goals, holdings, rates, comparisons, scenarios | FR-PROF/METR/EMRG/GOAL/HOLD/RATE/CMP/SCN |
| 04 | `stage/04-llm-gateway` | ModelGateway + LiteLLM + explain endpoint + grounding | FR-AI-*, NFR-AI-*, ADR-0005/0009 |
| 05 | `stage/05-ai-memory` | Three-scope memory, consent UI, context builder | FR-MEM-*, ADR-0007 |
| 06 | `stage/06-guardrails-evals` | Guardrails, refusal table, eval harness + datasets | FR-GRD-*, FR-EVAL-*, ADR-0011 |
| 07 | `stage/07-observability` | OTel, Langfuse, redaction, dashboards, alerts | FR-OBS-*, NFR-OBS-*, ADR-0013 |
| 08 | `stage/08-frontend-deployment` | Next.js app + public deployment | S1–S15, ADR-0014 |
| 09 | `stage/09-production-hardening` | Hardening, runbooks, perf/security pass, launch | DoD release checklist |

## Dependency graph

```
00 ─► 01 ─► 02 ─► 03 ─► 04 ─► 05 ─► 06 ─► 07 ─► 08 ─► 09
                        └─────────┴──────┘
                     (04 can start after 02 for a stub payload,
                      05 and 06 both depend on 04)
```

## Effort shape (indicative, not a commitment)

| Stage | Relative size | Notes |
| --- | --- | --- |
| 00 | S | Documentation |
| 01 | M | Auth + infra is the slowest invisible work |
| 02 | M | Highest test density |
| 03 | L | Most endpoints and UI-less surface |
| 04 | M | Gateway + grounding |
| 05 | S | Small, well-bounded |
| 06 | M | Evals take real effort |
| 07 | S | Wiring + redaction |
| 08 | L | Frontend + deploy |
| 09 | M | Hardening + docs |

## Milestones

| Milestone | Definition |
| --- | --- |
| M1 — Engine proven | Stage 02 complete: all golden cases + property tests green |
| M2 — Product usable | Stage 03 complete: a user can get metrics, comparisons and scenarios via API |
| M3 — AI grounded | Stage 06 complete: explanation pipeline passes determinism + injection suites |
| M4 — Public MVP | Stage 08 complete: public URL serving real users |
| M5 — Launch-ready | Stage 09 complete: release DoD satisfied |

## Rules

1. Stage N+1 cannot begin implementation before Stage N's gate passes.
2. A defect found in an earlier stage is fixed on a `fix/` branch and merged back; it does not wait.
3. Scope changes follow `docs/product/product-scope.md` §4.
4. Each stage ends with a written report in the same format as the Stage 00 report.
