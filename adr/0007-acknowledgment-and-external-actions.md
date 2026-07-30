# ADR-0007: Acknowledgment is a durable boundary; external actions are write-ahead

**Status:** Accepted · 2026-07-30 · Issue #1

## Decision

CharterOS acknowledges a command, receipt, artifact, or external action only after
its authoritative state and ledger event commit durably. PostgreSQL-backed writes
commit domain state, event, and outbox atomically. Artifact content is verified in
durable object storage before its metadata and event commit.

Every external action is write-ahead: intent, arguments hash, approval, deadline,
idempotency data, and reconciliation strategy commit before dispatch. A `running`
event commits immediately before the tool gateway crosses the external boundary.
If completion is not durably recorded before the deadline, the action becomes
`uncertain` and is reconciled by its declared strategy; it is never blindly
retried. Automatic replay is allowed only when the provider's idempotency contract
makes repeating the same key safe.

## Context

The guarantee “no lost acknowledged work” was underspecified. A database commit,
an object upload, and an HTTP side effect have different crash boundaries. In
particular, a remote action may succeed while its response is lost. Treating that
as failure can duplicate an email, purchase, merge, or payment; treating all
interrupted work as unrecoverable makes the system unusable.

## Consequences

- API success is returned only after the durable boundary; streaming progress is
  explicitly provisional.
- PostgreSQL and object-store durability settings are deployment conformance
  requirements, not tuning suggestions.
- Tool adapters must declare a reconciliation strategy before external dispatch.
- The chaos suite injects failure before and after every acknowledgment boundary,
  including the external-success/response-loss window.
- The guarantee covers state CharterOS or a conforming dependency acknowledged;
  it cannot strengthen a provider that lies about durability.
