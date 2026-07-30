# ADR-0005: Public protocol is JSON Schema for v1

**Status:** Accepted · 2026-07-30

## Decision

Event envelopes, API payloads, manifests, and receipts are specified with **JSON Schema** in the `protocol` package. Protobuf is revisited only if a second independent implementation demonstrates the need.

## Context

Every adjacent protocol CharterOS composes with — MCP, A2A agent cards, OpenAI-compatible APIs — is JSON-native. JSON Schema keeps the ledger human-readable (a stated product property: exported company records must be inspectable without tooling) and generates TypeScript and Python types adequately. Protobuf's wins (compactness, streaming performance) are not v1 bottlenecks; its cost (a second toolchain between the spec and every contributor) is immediate.

## Consequences

- Canonical serialization for event hashing must be specified explicitly (JCS / RFC 8785) since JSON has no canonical form by default.
- Schema versioning follows the envelope rule already in the spec: ignore unknown fields, reject unsupported major versions.
