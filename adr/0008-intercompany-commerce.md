# ADR-0008: Intercompany commerce is bilateral and settlement-neutral

**Status:** Proposed · 2026-07-30 · Issue #3

## Decision

CharterOS models paid work between organizations as a bilateral, versioned work
agreement. The buyer and seller are organizations; agents and humans act only as
authorized representatives. An agreement becomes effective when both organizations
sign the same canonical terms hash and any required funding condition is satisfied.

Milestone acceptance creates a payment obligation. It does not itself move funds.
Settlement executes through a pluggable provider under ADR-0007's external-action
journal. CharterOS core never takes custody, chooses a preferred rail, or charges a
transaction fee. Escrow, when required, is supplied by a declared third party or a
provider-specific conditional-payment mechanism.

Each party stores the same signed agreement envelope and cross-ledger receipts in
its own company ledger. Shared truth consists only of artifacts signed by the
required parties; unilateral local projections are not binding on the other party.

## Context

Internal budgets answer whether a company may spend. They do not identify a
counterparty, establish contractual terms, prove delegated authority, create an
invoice, or show that another company received settlement. Treating a remote agent
as a local contractor also erases the organization it represents. Payment rails
solve value transfer, not the agreement, acceptance, dispute, and evidence layers
that make a transfer justified.

## Consequences

- Organization identity and representative authority precede negotiation.
- Amounts use exact atomic units; floating-point money is forbidden.
- Agreements, revisions, signatures, submissions, acceptances, obligations,
  disputes, and settlement receipts are durable protocol objects.
- A2A may transport messages but is not the source of commercial authority.
- Tax, employment classification, sanctions, identity assurance, custody, and
  enforceability remain deployment- and jurisdiction-specific responsibilities.
- CharterOS records operational obligations and settlement evidence; it is not a
  general ledger. Accounting systems consume exports or events.

Detailed proposal: `docs/intercompany-commerce.md`.
