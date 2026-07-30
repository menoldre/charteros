# ADR-0006: Semantic search deferred; PostgreSQL full-text first

**Status:** Accepted · 2026-07-30

## Decision

Retrieval ships on PostgreSQL full-text search (`tsvector`, already in the schema). pgvector or an external vector store is added only when measured retrieval quality — the `context relevance and missing-context failures` metric — demands it, as an optional adapter.

## Context

Embeddings are explicitly non-authoritative in this design (spec §12): indexes are rebuilt from canonical records, never trusted as memory. Starting with FTS keeps the v0.1 dependency surface at exactly PostgreSQL + MinIO, and avoids shipping a second retrieval system before there is evidence the first one fails.

## Consequences

- The `memory` module's retrieval interface must be pluggable from day one so the adapter can arrive without API changes.
- Context-assembly quality metrics must land in v0.1 telemetry, or there will never be evidence to trigger the upgrade.
