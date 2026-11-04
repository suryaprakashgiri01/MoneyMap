# ADR-0007 — Structured Memory First; pgvector Deferred

**Status:** Accepted · **Date:** 2026-11-04 · **Stage:** 00

## Context

The spec allows pgvector "when semantic retrieval is required". For MVP, memory consists of a handful
of explicit user preferences, org planning assumptions, and curated generic knowledge terms. Every
access pattern is a known key lookup.

## Decision

1. Memory is a relational table (`memory_entries`) with explicit `scope`, `kind`, `key`, `provenance`.
2. Retrieval is deterministic: `WHERE user_id/org_id + kind ORDER BY updated_at DESC LIMIT n`.
3. No embeddings, no vector index, no similarity threshold in MVP.
4. Re-open only with a written requirement plus evidence: > ~100 memory rows per user **and** a
   measured case where key-based retrieval fails (ADR amendment required).

## Consequences

- Zero embedding cost, zero provider dependency for memory, trivially testable retrieval.
- Paraphrase-insensitive recall is not supported; acceptable because writes are explicit and keyed.
- If semantic recall is later added, the table gains an `embedding vector(1536)` column and a parallel
  retrieval path — no schema restructure.

## Alternatives considered

- **pgvector from day one** — rejected: cost, complexity, and eval surface without a requirement.
- **Conversation transcript memory** — rejected: unbounded PII retention, non-auditable, contradicts
  the memory safety rules.
- **External memory SaaS** — rejected: third-party financial-data exposure.

## Compliance

- SPEC-ARCH-004, FR-MEM-01..05, AC-MEM-01..04.
