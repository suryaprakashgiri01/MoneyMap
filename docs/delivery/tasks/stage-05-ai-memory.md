# Stage 05 — AI Memory

**Branch:** `stage/05-ai-memory`
**Spec:** FR-MEM-01..05, SPEC-ARCH-004, ADR-0007, AC-MEM-01..04

## Objective

Explicit, scoped, deletable memory that improves explanations without becoming a transcript store.

## Tasks

| ID | Task | Notes |
| --- | --- | --- |
| T05-01 | `memory_entries` table + migration | scope CHECK constraints |
| T05-02 | Memory validator | secrets, account numbers, injection patterns, length |
| T05-03 | Create/list/update/delete endpoints | explicit confirm path only |
| T05-04 | Context builder | deterministic key lookup, caps, scope assertions |
| T05-05 | Prompt injection of memory | fenced data block, ≤ 6 items, ≤ 200 chars each |
| T05-06 | Generic memory seed | curated terminology + product rules with `source_ref` |
| T05-07 | Deletion propagation test | deleted key absent from next bundle |
| T05-08 | Isolation tests | cross-user, cross-org → empty |
| T05-09 | Audit events | write/delete without content |

## Exit criteria

- No memory row can be created without an explicit user action (test proves zero rows after a
  calculation + explanation flow).
- Secret/account-number patterns rejected with a field-level error.
- Cross-org retrieval returns empty and logs a security event.
- Retrieval caps enforced and asserted.

## Out of scope

pgvector, embeddings, automatic extraction, chat history.

## Risks

| Risk | Mitigation |
| --- | --- |
| Scope drift into "auto-learn" | no extraction code path exists; ADR-0007 + review |
| Memory text steering the model | fenced as data + injection eval cases (Stage 06) |
| Stale preference overriding user intent | user-visible list + immediate deactivation |
