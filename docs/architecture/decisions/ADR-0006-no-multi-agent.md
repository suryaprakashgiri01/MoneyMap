# ADR-0006 — Custom Linear Orchestration; No Agents or LangGraph

**Status:** Accepted · **Date:** 2026-11-04 · **Stage:** 00

## Context

The AI workflow is a fixed 10-step pipeline (validate → scope-gate → load result → assemble context →
prompt → model → validate → ground → guard → trace). Agent frameworks promise flexibility for
multi-step reasoning, at the cost of non-determinism, debugging difficulty, cost, and a larger
security surface.

## Decision

Implement orchestration as a plain async function in `ai/orchestrator.py`:

- No agent loop, no planner, no self-critique cycle, no tool selection by the model.
- Steps are explicit, ordered, individually testable, and individually traced.
- The model receives one request and returns one response per pipeline run (one bounded retry).
- LangGraph is only reconsidered if a documented requirement demands branching over heterogeneous
  state with retries, *and* the pipeline form proves insufficient.

## Consequences

- Every failure is attributable to a numbered step.
- Adding a workflow means adding a prompt template + a template-specific payload pruner, not a graph.
- We forgo capabilities we do not need (dynamic tool use, multi-hop reasoning over arbitrary data).

## Alternatives considered

- **LangGraph** — rejected for MVP: dependency weight, implicit control flow, harder evaluation.
- **CrewAI / multi-agent** — rejected: no requirement justifies it; introduces unpredictable
  delegations and higher cost.
- **Direct inline code in the route handler** — rejected: untestable and mixes transport with policy.

## Compliance

- "Complex multi-agent architectures without a documented requirement" remains out of scope
  (`product-scope.md` §3.9).
