# ADR-0004: Live-event transport is SSE first, WebSocket later

**Status:** Accepted · 2026-07-30

## Decision

The live event stream (`GET /v1/companies/{id}/stream`) ships as **Server-Sent Events**. WebSocket is an additive upgrade if bidirectional needs emerge; it will never be required for core function.

## Context

The stream is a projection of an append-only ledger: strictly server-to-client, resumable by sequence number. SSE gives resumability (`Last-Event-ID` maps directly to ledger sequence), plain-HTTP operability through proxies and auth layers, and trivial client implementation for both humans' browsers and agents' HTTP clients. Commands already travel over REST; there is no bidirectional requirement in v1.

## Consequences

- Every consumer must tolerate reconnect-and-replay from a sequence number — which is the ledger consumption model anyway.
- If steering interactions later need a duplex channel, that is a new endpoint, not a change to the stream contract.
