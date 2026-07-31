# Intercompany Commerce Protocol

**Status:** Draft for review, revision 3 (audit fixes applied)<br>
**Related decision:** ADR-0008 (Proposed)<br>
**Tracking issue:** #3<br>
**Target:** Post-v1 protocol and schema extension

## 1. Purpose

This proposal defines paid gig work between two organizations running CharterOS or
compatible systems. It covers the commercial control plane around a payment:

- who the organizations are;
- who may bind each organization;
- what work was offered and accepted;
- which exact revision governs;
- what deliverables and verification are required;
- when a payment obligation arises;
- how funding, settlement, refund, and reconciliation are recorded;
- what happens when the parties disagree;
- how both organizations retain independently verifiable records.

It does not define a marketplace, payment network, cryptocurrency, tax engine,
employment-classification service, or court. Those systems integrate at explicit
boundaries.

## 2. Non-negotiable invariants

1. **Organizations are the counterparties.** An agent or human may represent an
   organization but is never silently substituted for it as buyer or seller.
2. **Authority is ledgered.** A message or agent card alone cannot bind a company.
   The representative must present a current, scoped authorization proof.
3. **Signatures bind exact bytes.** Both parties sign the same RFC 8785 canonical
   agreement revision. A signature over one revision cannot accept another.
4. **Negotiation is not commitment.** Offers and counteroffers are non-binding
   until the agreement's activation conditions are satisfied.
5. **Acceptance is not settlement.** Acceptance creates an obligation; a separate
   governed action settles it.
6. **Money is exact.** Amounts are integers in declared atomic units. Binary
   floating point is forbidden.
7. **No blind payment retry.** Every settlement follows ADR-0007 and declares how
   an ambiguous result will be reconciled before dispatch.
8. **Neither ledger is sovereign over both parties.** Only mutually signed facts
   and valid protocol receipts form shared state.
9. **Disputes preserve evidence.** Opening a dispute never deletes or rewrites the
   agreement, submission, verification, or settlement history.
10. **CharterOS takes no toll.** Core protocol behavior cannot depend on a fee paid
    to the CharterOS project or operator.

## 3. Boundaries

### 3.1 CharterOS is responsible for

- agreement negotiation and versioning;
- representative authorization checks;
- work and milestone state;
- deliverable hashes and verification evidence;
- acceptance and payment-obligation creation;
- funding and settlement orchestration;
- cross-ledger signed messages and receipts;
- dispute workflow and resolution records;
- auditable export to accounting, legal, and compliance systems.

### 3.2 CharterOS is not

- a party to the agreement;
- a bank, money transmitter, escrow company, or custodian merely because this
  protocol is installed;
- a source of legal identity by itself;
- a general ledger or tax ledger;
- an arbiter unless an agreement explicitly names a separate operator in that role;
- a guarantor that a counterparty, provider, or external identity issuer is honest.

Deployments that custody funds or operate regulated payment services require their
own legal and compliance analysis. Core CharterOS should prefer non-custodial
coordination and licensed third-party providers.

## 4. Parties and identity

### 4.1 Organization identity

An intercompany party record contains:

- a globally stable organization identifier;
- display and legal names, when disclosed;
- service endpoints;
- signing keys and rotation history;
- identity claims with issuer, assurance level, issue time, and expiry;
- payment and invoice endpoints;
- supported protocol versions and settlement capabilities;
- local trust status and risk notes.

The identifier is opaque to CharterOS core. A deployment may use a DID, verified
domain identity, registry identifier, or another scheme. The protocol records the
identifier scheme and verification evidence rather than claiming all schemes are
equivalent.

An A2A Signed Agent Card may help discover an agent endpoint. It does not, by
itself, prove that the agent may sign contracts or spend for an organization.

### 4.2 Representative authority

Every binding message carries an authorization proof with:

- organization identifier;
- representative principal identifier;
- permitted action, such as `agreement.propose`, `agreement.sign`,
  `milestone.submit`, `milestone.accept`, `dispute.resolve`, or `payment.release`;
- resource scope;
- monetary ceiling when relevant;
- valid-from and expiry times;
- issuing authority and signature;
- revocation reference or status endpoint.

The receiving company verifies the proof at message receipt and stores both the
proof and verification result. Later revocation does not erase an action that was
valid when accepted, unless the agreement says otherwise.

Revocation and other later restrictions are recorded as appended status
observations against the stored proof (section 16); the stored proof row itself is
immutable, and its recorded verification result is the result at receipt time.
Status observations are monotonic restrictions: `suspended`, `expired`, and
`revoked` can never restore a proof to `valid`. Resuming authority requires a new
proof. Effective status is the most restrictive valid observation recorded by the
receiver, not whichever remote timestamp sorts last. Every binding message is
evaluated against effective status at the moment of receipt.

Any restriction denies use of that proof. For display and audit, deterministic
precedence is `revoked` over `expired` over `suspended`; precedence never permits an
action. Equal or conflicting issuer observations are retained and quarantine the
proof until verified.

### 4.3 Local counterparty records

Each company keeps a local record for the remote organization. Local trust status,
risk classification, blocks, annotations, and internal due diligence are private
projections and are never presented as mutually agreed facts.

## 5. Agreement model

### 5.1 Agreement envelope

```json
{
  "protocol": "charteros-commerce/v1alpha1",
  "agreementId": "0191a22c-20db-7bf1-aadc-6c018e620e31",
  "revision": 3,
  "previousRevisionHash": "sha256:...",
  "buyerOrganizationId": "org:example:buyer",
  "sellerOrganizationId": "org:example:seller",
  "terms": {
    "title": "Accessibility review of the public web application",
    "scope": "...",
    "governingTerms": { "document": "artifact:..." },
    "confidentiality": { "profile": "mutual-basic-v1" },
    "milestones": [],
    "disputePolicy": {},
    "terminationPolicy": {},
    "activationConditions": []
  },
  "createdAt": "2026-07-30T20:00:00Z",
  "expiresAt": "2026-08-06T20:00:00Z"
}
```

The `terms` document is extensible JSON validated by a versioned JSON Schema.
Unknown optional fields are preserved. Unknown required capabilities cause
rejection, not best-effort interpretation.

### 5.2 Revision rules

- Revisions are immutable and numbered from one.
- A new revision names the previous revision hash.
- Only one revision may be active.
- Any material change creates a new revision and requires both signatures again.
- A revision expires if it is not activated before `expiresAt`.
- Concurrent counteroffers form branches. The parties must explicitly choose one;
  revision number alone never resolves a fork.
- The binding revision is identified by its canonical SHA-256 hash, not merely its
  number.

### 5.3 Activation

Activation is a signed prepare protocol, not a cross-ledger transaction:

1. Each ledger independently verifies both agreement signatures, representative
   authority, organization and key status, activation conditions, and required
   funding evidence.
2. Each party constructs the same activation snapshot: agreement revision hash,
   ordered activation-condition results, ordered funding commitment hashes, buyer
   and seller organization IDs, and protocol version.
3. Each authorized party signs `activation.prepared` over the snapshot hash and
   sends it to the other party.
4. A ledger marks the agreement `active` only after it holds one valid buyer
   preparation and one valid seller preparation over the identical snapshot hash.
   Activation is then a deterministic local derivation; neither party waits for an
   additional “I activated” acknowledgment.
5. Each ledger emits `agreement.activated` and sends a receipt as evidence. A lost
   receipt is redelivered idempotently and cannot undo activation.

Preparations over different snapshot hashes do not activate anything. Either party
may issue a new preparation only after the changed funding or conditions produce a
new snapshot; earlier preparations remain evidence but cannot combine with the new
hash. This avoids both impossible cross-database atomicity and a two-party
acknowledgment deadlock.

## 6. Agreement lifecycle

```text
draft
  -> offered
  -> negotiating
  -> signed
  -> awaiting_funding
  -> active
  -> completed
  -> closed

draft | offered | negotiating | signed | awaiting_funding -> expired | cancelled
active -> suspended | disputed | terminated
disputed -> active | terminated | completed
```

`signed` means both signatures exist. `active` means all activation conditions are
satisfied. `completed` means every required milestone reached a terminal outcome
and all resulting obligations are resolved. `closed` means the retention clock has
started; it does not mean records may be erased.

## 7. Milestones and work

Each agreement contains one or more milestones. A milestone defines:

- title, scope, and delivery deadline;
- price and asset;
- exact deliverables;
- acceptance criteria and verification methods;
- buyer review window;
- whether silence can ever imply acceptance; default: no;
- allowed revision count and rework policy;
- funding requirement;
- partial-acceptance policy;
- cancellation and kill-fee terms;
- dependencies on other milestones;
- who bears provider and settlement fees.

The seller creates local tasks to perform a milestone. The buyer may create local
review tasks. Those tasks remain private unless deliberately shared. The bilateral
protocol exchanges submissions, evidence, verification results, acceptance, and
rejection—not either company's internal reasoning or private task graph.

### 7.1 Milestone lifecycle

```text
pending -> ready -> in_progress -> submitted -> under_review -> accepted -> payable
                        ^              |
                        |              v
                        +---------- changes_requested

pending | ready | in_progress -> cancelled
submitted | under_review -> disputed
payable -> settling -> settled
```

### 7.2 Submission

A submission is immutable and contains:

- milestone and agreement revision;
- seller organization and authorized representative;
- ordered deliverable manifest;
- content hashes, media types, sizes, and retrieval grants;
- verification performed by the seller;
- known limitations;
- submission time;
- signature over the canonical submission envelope.

A revised submission supersedes but does not mutate the previous submission.

### 7.3 Acceptance

The buyer evaluates the agreement's acceptance criteria through CharterOS's
verification ladder. An acceptance or rejection:

- identifies the exact submission hash;
- records a result for every required criterion;
- links verification evidence;
- is signed by an authorized buyer representative;
- records the policy and agreement revision in force;
- is immutable.

Acceptance commits atomically in the buyer's ledger together with the payment
obligation, the local ledger event, and the outbound bilateral message. The
seller's mirrored obligation is created when that message is applied and
acknowledged: it is eventually consistent through at-least-once delivery and a
signed receipt, never assumed. Two independent ledgers cannot share one
transaction, and this protocol does not pretend otherwise. Acceptance never
invokes a payment provider in the same transaction.

## 8. Monetary model

### 8.1 Amounts and assets

```json
{
  "asset": "iso4217:USD",
  "amountAtomic": "125000",
  "exponent": 2
}
```

- `amountAtomic` is a base-10 integer encoded as a string on the wire and
  `numeric(78,0)` in PostgreSQL.
- `exponent` is captured in the signed terms and cannot change retroactively.
- Asset identifiers are namespaced. CharterOS does not infer interchangeability
  between assets with similar symbols.
- Negative payment obligations are forbidden. Refunds and credits are separate
  records with explicit direction.
- Currency conversion requires a signed quote specifying source asset, destination
  asset, amounts, rate source, fees, and expiry. A dashboard exchange rate cannot
  alter an agreement.

### 8.2 Payment obligation

A payment obligation records:

- debtor organization;
- creditor organization;
- agreement, milestone, submission, and acceptance hashes;
- exact amount and asset;
- due time;
- settlement policy;
- fee allocation;
- status and remaining amount;
- funding evidence;
- invoice references when applicable.

Obligations are append-only economic facts with a mutable projection of their
current state. Adjustments use credit, debit, waiver, or replacement records; they
never edit the original amount.

### 8.3 Internal budgets versus obligations

Internal budget reservation answers, “May this company commit or spend this
amount?” A payment obligation answers, “What does this company owe the other
company under the signed agreement?” They are linked but never conflated.

The buyer reserves internal budget before signing when policy requires it. Funding
or escrow evidence may be an activation condition. Acceptance converts the
commercial commitment into a payable obligation; settlement converts that
obligation into a recorded transfer result.

## 9. Funding and escrow

CharterOS supports three funding modes:

1. **Unfunded promise.** The buyer pays after acceptance. Appropriate only when
   counterparty trust and policy allow it.
2. **Provider authorization or conditional payment.** A payment provider proves
   funds are reserved or conditionally payable without CharterOS taking custody.
3. **Third-party escrow.** A separately identified provider holds or controls funds
   under its own terms and regulatory responsibilities.

Funding evidence names provider, amount, asset, agreement revision, milestone,
expiry, conditions, external reference, and evidence hash. “Funded” is never a
free-form status asserted by an agent.

A funding commitment attaches to one agreement and, when scoped, one milestone
because activation-condition funding exists before any payment obligation does.
The immutable commitment fixes its global identifier, mode, provider, amount,
asset, and scope. Append-only observations record verification, expiry, release,
refund, invalidation, and the later obligation linkage. An observation cannot
change the commitment's economic identity. When acceptance creates an obligation,
an `obligation_linked` observation may connect it only to a commitment for the same
agreement, milestone, amount, and asset.

Observations are sequenced per commitment by the receiving ledger. Current state is
a deterministic fold of that sequence under the funding state machine, not the row
with the greatest wall-clock timestamp. Duplicate evidence returns the prior
observation; conflicting evidence quarantines the commitment.

Core CharterOS does not pool funds, maintain omnibus balances, or hold private keys
capable of moving customer funds outside a task-scoped settlement grant.

## 10. Settlement

### 10.1 Provider interface

```typescript
interface SettlementProvider {
  capabilities(): Promise<SettlementCapabilities>;
  quote(request: SettlementQuoteRequest): Promise<SignedSettlementQuote>;
  proveFunding(request: FundingRequest): Promise<FundingEvidence>;
  settle(request: SettlementRequest): Promise<SettlementReceipt>;
  refund(request: RefundRequest): Promise<SettlementReceipt>;
  reconcile(reference: ReconciliationReference): Promise<SettlementObservation>;
}
```

Provider capabilities declare supported assets, funding modes, idempotency,
reconciliation, finality semantics, fee behavior, refund support, and required
identity attributes. An adapter cannot claim a feature the provider does not
contractually supply.

### 10.2 Settlement lifecycle

```text
created -> approval_pending -> ready -> dispatching -> submitted
submitted -> pending_finality -> settled
dispatching | submitted | pending_finality -> reconciliation_required
created | approval_pending | ready -> cancelled
reconciliation_required -> submitted | settled | failed | disputed
```

Every settlement is an `irreversible` tool call unless a stricter deployment policy
applies. It follows ADR-0007:

1. persist obligation, approval, quote, provider, exact request hash,
   idempotency data, deadline, and reconciliation strategy;
2. commit `payment.started` before dispatch;
3. call the provider once;
4. persist the result and `payment.settled`, `payment.failed`, or
   `payment.reconciliation_required`;
5. send the signed receipt to the counterparty;
6. wait for a counterparty receipt acknowledgment.

Provider success and settlement finality are distinct. The adapter defines when a
transfer is reversible, pending, or final, and records the evidence supporting that
classification.

### 10.3 Partial settlement and fees

- An obligation may have multiple settlement attempts and receipts.
- The sum of final successful principal amounts plus explicit waivers cannot exceed
  the obligation amount.
- Provider fees are separate from principal and assigned according to the signed
  agreement.
- Overpayment creates a refund or credit obligation; it is not silently applied to
  unrelated work.
- Settlement in a different asset requires the signed conversion quote permitted
  by the agreement.

## 11. Invoices and accounting integration

An invoice is an optional commercial document required by agreement or deployment
policy. Issued invoices are immutable. Corrections use a replacement invoice,
credit note, or debit note linked to the original.

CharterOS records operational obligations and settlement proofs but does not claim
to maintain generally accepted accounting books. It emits events and exports:

- agreement and counterparty references;
- invoice and credit-note documents;
- obligation creation and adjustment;
- funding and escrow evidence;
- principal, fees, asset, and conversion quote;
- settlement and refund receipts;
- dispute outcomes.

An accounting integration determines chart-of-account mappings, recognition rules,
tax treatment, and reporting jurisdiction.

## 12. Disputes

### 12.1 Dispute policy

The signed agreement states:

- who may open a dispute;
- eligible subjects and filing deadline;
- whether disputed funds remain reserved;
- negotiation period;
- named mediator or arbiter, if any;
- evidence rules;
- possible remedies;
- fee allocation;
- fallback if the arbiter is unavailable.

CharterOS supplies workflow, evidence integrity, and authority checks. It does not
invent an arbiter after a dispute begins.

### 12.2 Dispute lifecycle

```text
open -> response_due -> negotiating -> resolved
                         |            |
                         v            v
                     escalated -> adjudicated
open | response_due | negotiating -> withdrawn
```

Opening a valid dispute suspends settlement only for the disputed amount and only
when the agreement says it should. Undisputed obligations may continue.

### 12.3 Resolutions

A resolution may:

- accept or reject the submission;
- require rework;
- split a milestone;
- reduce, waive, or create an obligation;
- authorize full or partial release;
- authorize refund or compensation;
- terminate the agreement.

The resolution is signed by the parties or the previously named adjudicator and is
stored in both ledgers. Monetary changes are new adjustment records, never edits.

## 13. Cross-ledger protocol

### 13.1 Message envelope

```json
{
  "protocol": "charteros-commerce/v1alpha1",
  "messageId": "0191a2d8-30fb-7ca0-a307-e53167b73090",
  "messageType": "milestone.submitted",
  "agreementId": "0191a22c-20db-7bf1-aadc-6c018e620e31",
  "agreementRevisionHash": "sha256:...",
  "senderOrganizationId": "org:example:seller",
  "recipientOrganizationId": "org:example:buyer",
  "representative": "principal:...",
  "authorizationProof": {},
  "sentAt": "2026-08-02T15:04:05Z",
  "expiresAt": null,
  "payload": {},
  "payloadHash": "sha256:...",
  "signature": {}
}
```

The envelope and payload are canonicalized separately. The signature covers the
envelope fields and payload hash. Large or confidential artifacts travel out of
band through scoped retrieval grants; the message carries immutable hashes.

### 13.2 Delivery

- Messages are at-least-once and idempotent by `(senderOrganizationId, messageId)`.
- The receiver verifies protocol version, signature, organization identity,
  representative authority, agreement revision, expiry, and payload schema before
  applying a command.
- Receipt acknowledgments name the message ID, receiver ledger event ID, result,
  and result hash.
- A transport-level HTTP success is not a commercial acknowledgment.
- A2A, HTTPS webhooks, or another authenticated transport may carry the envelope.
  Transport choice does not alter its meaning.
- Outbound messages remain in the transactional outbox until a valid receipt or a
  terminal delivery policy outcome exists.

### 13.3 Shared and local truth

Mutually binding facts require the signatures defined by the agreement. Examples:

- active agreement revision: buyer plus seller;
- seller submission: seller, then buyer receipt acknowledgment;
- acceptance or rejection: buyer, then seller receipt acknowledgment;
- negotiated dispute resolution: buyer plus seller;
- adjudicated resolution: named adjudicator;
- settlement: provider receipt plus payer record, then payee acknowledgment.

Local facts—risk flags, internal approvals, private verification, budgets, task
graphs—remain local. A company may disclose their hashes as evidence without
disclosing their contents.

### 13.4 Conflicts

- Highest revision number does not win; the latest mutually signed revision hash
  does.
- Duplicate messages return the prior receipt.
- Concurrent counteroffers remain branches until both parties select one.
- A submission against an inactive revision is rejected.
- A settlement for an unknown or non-payable obligation is quarantined rather than
  auto-applied.
- Clock disagreement never chooses a winner. Signatures, revisions, and explicit
  deadlines govern; timestamps are evidence, not consensus.

## 14. Privacy and selective disclosure

- Agreement envelopes contain only data both parties require.
- Private company state is referenced by hashes or signed attestations where
  possible.
- Artifact retrieval grants are scoped to agreement, recipient, purpose, and time.
- Secret values, payment credentials, private model context, and internal policy
  documents never enter bilateral messages.
- Redaction produces a new export view and a ledgered disclosure event; it does not
  rewrite signed source records.
- Retention and deletion conflicts are governed by the agreement and applicable
  deployment policy; the protocol must not promise that another party deletes its
  independently held evidence.

## 15. Failure handling

| Failure | Required behavior |
|---|---|
| Representative loses authority during negotiation | Reject later binding messages; retain prior non-binding history |
| Authority revoked after a valid signature | Preserve the signature-time fact; apply termination rules if required |
| One ledger derives activation and the other does not | Exchange missing signed preparations; receipts aid diagnosis but never gate activation |
| Seller disappears after funding | Apply deadline, cancellation, and escrow terms |
| Buyer disappears after submission | Apply review deadline and dispute policy; silence is not acceptance unless signed terms explicitly say so |
| Payment succeeds but response is lost | Enter reconciliation; follow the declared provider strategy; never blind-retry |
| Provider reports success but seller cannot observe funds | Record disagreement, verify provider finality evidence, and open dispute if unresolved |
| Settlement rail becomes unavailable | Keep obligation open; choose another rail only through an allowed signed amendment or settlement policy |
| Duplicate settlement message arrives | Return the existing receipt without creating another transfer |
| Agreement branches during negotiation | Neither branch activates until both parties sign the same hash |

## 16. Proposed PostgreSQL schema

This schema is a design draft, not an accepted migration. It assumes the core schema
from the project specification and preserves tenant-composite foreign keys, RLS,
and append-only evidence. Tables are local projections: the same bilateral
agreement normally appears once in each party's ledger under a different
`company_id` but the same `global_agreement_id`.

```sql
BEGIN;
SET search_path TO charteros, public;

-- Required before tenant-composite references can target principals. This should
-- become part of the core schema before this extension is implemented.
ALTER TABLE principals
  ADD CONSTRAINT principals_id_company_unique UNIQUE (id, company_id);

ALTER TABLE events
  ADD CONSTRAINT events_id_company_unique UNIQUE (id, company_id);

CREATE TABLE commerce_counterparties (
  id                    uuid PRIMARY KEY,
  company_id            uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  organization_id       text NOT NULL,
  identifier_scheme     text NOT NULL,
  display_name          text NOT NULL,
  legal_name            text,
  protocol_endpoint     text,
  invoice_endpoint      text,
  status                text NOT NULL DEFAULT 'unverified'
                        CHECK (status IN ('unverified', 'verified', 'restricted', 'blocked', 'retired')),
  identity_assurance    jsonb NOT NULL DEFAULT '{}'::jsonb,
  settlement_capabilities jsonb NOT NULL DEFAULT '[]'::jsonb,
  metadata              jsonb NOT NULL DEFAULT '{}'::jsonb,
  created_at            timestamptz NOT NULL DEFAULT clock_timestamp(),
  updated_at            timestamptz NOT NULL DEFAULT clock_timestamp(),
  UNIQUE (company_id, organization_id),
  UNIQUE (id, company_id),
  CHECK (jsonb_typeof(identity_assurance) = 'object'),
  CHECK (jsonb_typeof(settlement_capabilities) = 'array'),
  CHECK (jsonb_typeof(metadata) = 'object')
);

CREATE TRIGGER commerce_counterparties_set_updated_at
BEFORE UPDATE ON commerce_counterparties
FOR EACH ROW EXECUTE FUNCTION set_updated_at();

CREATE TABLE commerce_counterparty_keys (
  id                    uuid PRIMARY KEY,
  company_id            uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  counterparty_id       uuid NOT NULL,
  key_id                text NOT NULL,
  algorithm             text NOT NULL,
  public_key_material   text NOT NULL,
  valid_from            timestamptz NOT NULL,
  valid_until           timestamptz,
  revoked_at            timestamptz,
  verification_evidence jsonb NOT NULL DEFAULT '{}'::jsonb,
  created_at            timestamptz NOT NULL DEFAULT clock_timestamp(),
  UNIQUE (counterparty_id, key_id),
  UNIQUE (id, company_id),
  FOREIGN KEY (counterparty_id, company_id)
    REFERENCES commerce_counterparties(id, company_id) ON DELETE CASCADE,
  CHECK (valid_until IS NULL OR valid_until > valid_from),
  CHECK (jsonb_typeof(verification_evidence) = 'object')
);

CREATE TABLE commerce_authorization_proofs (
  id                    uuid PRIMARY KEY,
  company_id            uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  counterparty_id       uuid NOT NULL,
  organization_id       text NOT NULL,
  representative_id     text NOT NULL,
  actions               jsonb NOT NULL,
  resource_scope        text NOT NULL,
  monetary_limit        numeric(78,0),
  asset_id              text,
  valid_from            timestamptz NOT NULL,
  expires_at            timestamptz NOT NULL,
  issuer_key_id         text NOT NULL,
  proof_document        jsonb NOT NULL,
  proof_sha256          text NOT NULL CHECK (proof_sha256 ~ '^[0-9a-f]{64}$'),
  verification_status   text NOT NULL CHECK (verification_status IN ('valid', 'invalid', 'unknown', 'expired', 'revoked')),
  verified_at           timestamptz,
  created_at            timestamptz NOT NULL DEFAULT clock_timestamp(),
  UNIQUE (id, company_id),
  FOREIGN KEY (counterparty_id, company_id)
    REFERENCES commerce_counterparties(id, company_id) ON DELETE RESTRICT,
  CHECK (monetary_limit IS NULL OR monetary_limit >= 0),
  CHECK ((monetary_limit IS NULL) = (asset_id IS NULL)),
  CHECK (expires_at > valid_from),
  CHECK (jsonb_typeof(actions) = 'array'),
  CHECK (jsonb_typeof(proof_document) = 'object')
);

-- Proof rows are immutable; verification_status is the result at receipt time.
-- Later restrictions are append-only and monotonic. A restricted proof is never
-- reinstated; renewed authority requires a new proof.
CREATE TABLE commerce_authorization_proof_statuses (
  id                    uuid PRIMARY KEY,
  company_id            uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  authorization_proof_id uuid NOT NULL,
  sequence              bigint NOT NULL CHECK (sequence > 0),
  status                text NOT NULL CHECK (status IN ('suspended', 'revoked', 'expired')),
  source                text NOT NULL CHECK (source IN (
                          'issuer_notice', 'status_endpoint', 'local_expiry', 'local_policy'
                        )),
  issuer_key_id         text,
  evidence              jsonb NOT NULL DEFAULT '{}'::jsonb,
  evidence_sha256       text NOT NULL CHECK (evidence_sha256 ~ '^[0-9a-f]{64}$'),
  signature             jsonb,
  effective_at          timestamptz NOT NULL,
  received_at           timestamptz NOT NULL DEFAULT clock_timestamp(),
  created_at            timestamptz NOT NULL DEFAULT clock_timestamp(),
  UNIQUE (authorization_proof_id, sequence),
  UNIQUE (authorization_proof_id, evidence_sha256),
  UNIQUE (id, company_id),
  FOREIGN KEY (authorization_proof_id, company_id)
    REFERENCES commerce_authorization_proofs(id, company_id) ON DELETE CASCADE,
  CHECK (jsonb_typeof(evidence) = 'object'),
  CHECK (signature IS NULL OR jsonb_typeof(signature) = 'object'),
  CHECK (
    (source IN ('issuer_notice', 'status_endpoint')
      AND issuer_key_id IS NOT NULL AND signature IS NOT NULL)
    OR
    (source IN ('local_expiry', 'local_policy')
      AND issuer_key_id IS NULL AND signature IS NULL)
  )
);

CREATE INDEX commerce_authorization_proof_statuses_latest_idx
  ON commerce_authorization_proof_statuses (authorization_proof_id, sequence DESC);

CREATE TABLE commerce_agreements (
  id                    uuid PRIMARY KEY,
  company_id            uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  global_agreement_id   uuid NOT NULL,
  local_role            text NOT NULL CHECK (local_role IN ('buyer', 'seller')),
  counterparty_id       uuid NOT NULL,
  status                text NOT NULL DEFAULT 'draft'
                        CHECK (status IN (
                          'draft', 'offered', 'negotiating', 'signed',
                          'awaiting_funding', 'active', 'suspended', 'disputed',
                          'completed', 'terminated', 'cancelled', 'expired', 'closed'
                        )),
  current_revision      integer NOT NULL DEFAULT 0 CHECK (current_revision >= 0),
  active_revision_hash  text CHECK (active_revision_hash IS NULL OR active_revision_hash ~ '^[0-9a-f]{64}$'),
  title                 text NOT NULL,
  created_by            uuid NOT NULL,
  created_at            timestamptz NOT NULL DEFAULT clock_timestamp(),
  updated_at            timestamptz NOT NULL DEFAULT clock_timestamp(),
  activated_at          timestamptz,
  closed_at             timestamptz,
  UNIQUE (company_id, global_agreement_id),
  UNIQUE (id, company_id),
  FOREIGN KEY (counterparty_id, company_id)
    REFERENCES commerce_counterparties(id, company_id) ON DELETE RESTRICT,
  FOREIGN KEY (created_by, company_id)
    REFERENCES principals(id, company_id) ON DELETE RESTRICT
);

CREATE TRIGGER commerce_agreements_set_updated_at
BEFORE UPDATE ON commerce_agreements
FOR EACH ROW EXECUTE FUNCTION set_updated_at();

CREATE TABLE commerce_agreement_revisions (
  id                    uuid PRIMARY KEY,
  company_id            uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  agreement_id          uuid NOT NULL,
  revision              integer NOT NULL CHECK (revision > 0),
  parent_revision_hash  text CHECK (parent_revision_hash IS NULL OR parent_revision_hash ~ '^[0-9a-f]{64}$'),
  proposer_organization_id text NOT NULL,
  terms_document        jsonb NOT NULL,
  canonical_sha256      text NOT NULL CHECK (canonical_sha256 ~ '^[0-9a-f]{64}$'),
  offered_at            timestamptz NOT NULL,
  expires_at            timestamptz,
  created_at            timestamptz NOT NULL DEFAULT clock_timestamp(),
  UNIQUE (agreement_id, canonical_sha256),
  UNIQUE (id, company_id),
  FOREIGN KEY (agreement_id, company_id)
    REFERENCES commerce_agreements(id, company_id) ON DELETE CASCADE,
  CHECK (jsonb_typeof(terms_document) = 'object'),
  CHECK (expires_at IS NULL OR expires_at > offered_at)
);

CREATE INDEX commerce_agreement_revisions_number_idx
  ON commerce_agreement_revisions (agreement_id, revision);

CREATE TABLE commerce_agreement_signatures (
  id                    uuid PRIMARY KEY,
  company_id            uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  agreement_revision_id uuid NOT NULL,
  signer_organization_id text NOT NULL,
  representative_id     text NOT NULL,
  authorization_proof_id uuid,
  key_id                text NOT NULL,
  algorithm             text NOT NULL,
  signature             text NOT NULL,
  canonical_sha256      text NOT NULL CHECK (canonical_sha256 ~ '^[0-9a-f]{64}$'),
  signed_at             timestamptz NOT NULL,
  verification_status   text NOT NULL CHECK (verification_status IN ('valid', 'invalid', 'revoked_at_signing')),
  verified_at           timestamptz,
  created_at            timestamptz NOT NULL DEFAULT clock_timestamp(),
  UNIQUE (agreement_revision_id, signer_organization_id, canonical_sha256),
  UNIQUE (id, company_id),
  FOREIGN KEY (agreement_revision_id, company_id)
    REFERENCES commerce_agreement_revisions(id, company_id) ON DELETE CASCADE,
  FOREIGN KEY (authorization_proof_id, company_id)
    REFERENCES commerce_authorization_proofs(id, company_id) ON DELETE RESTRICT
);

CREATE UNIQUE INDEX commerce_agreement_signatures_one_valid_idx
  ON commerce_agreement_signatures (agreement_revision_id, signer_organization_id)
  WHERE verification_status = 'valid';

-- Each party signs the same activation snapshot independently. Possessing valid
-- buyer and seller preparations for one snapshot makes activation a deterministic
-- local derivation rather than an impossible cross-database commit.
CREATE TABLE commerce_activation_preparations (
  id                    uuid PRIMARY KEY,
  company_id            uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  agreement_revision_id uuid NOT NULL,
  snapshot_document     jsonb NOT NULL,
  snapshot_sha256       text NOT NULL CHECK (snapshot_sha256 ~ '^[0-9a-f]{64}$'),
  signer_organization_id text NOT NULL,
  representative_id     text NOT NULL,
  authorization_proof_id uuid,
  key_id                text NOT NULL,
  algorithm             text NOT NULL,
  signature             text NOT NULL,
  preparation_sha256    text NOT NULL CHECK (preparation_sha256 ~ '^[0-9a-f]{64}$'),
  prepared_at           timestamptz NOT NULL,
  verification_status   text NOT NULL CHECK (verification_status IN ('valid', 'invalid', 'revoked_at_signing')),
  verified_at           timestamptz,
  created_at            timestamptz NOT NULL DEFAULT clock_timestamp(),
  UNIQUE (agreement_revision_id, signer_organization_id, preparation_sha256),
  UNIQUE (id, company_id),
  FOREIGN KEY (agreement_revision_id, company_id)
    REFERENCES commerce_agreement_revisions(id, company_id) ON DELETE CASCADE,
  FOREIGN KEY (authorization_proof_id, company_id)
    REFERENCES commerce_authorization_proofs(id, company_id) ON DELETE RESTRICT,
  CHECK (jsonb_typeof(snapshot_document) = 'object')
);

CREATE UNIQUE INDEX commerce_activation_preparations_one_valid_idx
  ON commerce_activation_preparations (
    agreement_revision_id, snapshot_sha256, signer_organization_id
  )
  WHERE verification_status = 'valid';

CREATE TABLE commerce_milestones (
  id                    uuid PRIMARY KEY,
  company_id            uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  agreement_id          uuid NOT NULL,
  global_milestone_id   uuid NOT NULL,
  milestone_key         text NOT NULL,
  status                text NOT NULL DEFAULT 'pending'
                        CHECK (status IN (
                          'pending', 'ready', 'in_progress', 'submitted',
                          'under_review', 'changes_requested', 'accepted',
                          'payable', 'settling', 'settled', 'disputed', 'cancelled'
                        )),
  active_terms_id       uuid,
  created_at            timestamptz NOT NULL DEFAULT clock_timestamp(),
  updated_at            timestamptz NOT NULL DEFAULT clock_timestamp(),
  UNIQUE (agreement_id, milestone_key),
  UNIQUE (company_id, global_milestone_id),
  UNIQUE (id, company_id),
  FOREIGN KEY (agreement_id, company_id)
    REFERENCES commerce_agreements(id, company_id) ON DELETE CASCADE
);

CREATE TRIGGER commerce_milestones_set_updated_at
BEFORE UPDATE ON commerce_milestones
FOR EACH ROW EXECUTE FUNCTION set_updated_at();

-- Economic and acceptance terms are immutable and revision-specific. Lifecycle
-- state is kept separately in commerce_milestones so a status update cannot
-- accidentally rewrite the signed bargain.
CREATE TABLE commerce_milestone_terms (
  id                    uuid PRIMARY KEY,
  company_id            uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  milestone_id          uuid NOT NULL,
  agreement_revision_id uuid NOT NULL,
  sequence              integer NOT NULL CHECK (sequence > 0),
  title                 text NOT NULL,
  amount_atomic         numeric(78,0) NOT NULL CHECK (amount_atomic >= 0),
  asset_id              text NOT NULL,
  asset_exponent        smallint NOT NULL CHECK (asset_exponent BETWEEN 0 AND 30),
  deliverables          jsonb NOT NULL,
  acceptance_criteria   jsonb NOT NULL,
  due_at                timestamptz,
  review_deadline_seconds integer CHECK (review_deadline_seconds IS NULL OR review_deadline_seconds > 0),
  canonical_sha256      text NOT NULL CHECK (canonical_sha256 ~ '^[0-9a-f]{64}$'),
  created_at            timestamptz NOT NULL DEFAULT clock_timestamp(),
  UNIQUE (milestone_id, agreement_revision_id),
  UNIQUE (agreement_revision_id, sequence),
  UNIQUE (id, company_id),
  FOREIGN KEY (milestone_id, company_id)
    REFERENCES commerce_milestones(id, company_id) ON DELETE CASCADE,
  FOREIGN KEY (agreement_revision_id, company_id)
    REFERENCES commerce_agreement_revisions(id, company_id) ON DELETE CASCADE,
  CHECK (jsonb_typeof(deliverables) = 'array'),
  CHECK (jsonb_typeof(acceptance_criteria) = 'array')
);

ALTER TABLE commerce_milestones
  ADD CONSTRAINT commerce_milestones_active_terms_company_fk
  FOREIGN KEY (active_terms_id, company_id)
  REFERENCES commerce_milestone_terms(id, company_id) ON DELETE RESTRICT;

CREATE TABLE commerce_submissions (
  id                    uuid PRIMARY KEY,
  company_id            uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  milestone_id          uuid NOT NULL,
  submission_number     integer NOT NULL CHECK (submission_number > 0),
  agreement_revision_hash text NOT NULL CHECK (agreement_revision_hash ~ '^[0-9a-f]{64}$'),
  seller_organization_id text NOT NULL,
  representative_id     text NOT NULL,
  authorization_proof_id uuid,
  deliverable_manifest  jsonb NOT NULL,
  known_limitations     text,
  canonical_sha256      text NOT NULL CHECK (canonical_sha256 ~ '^[0-9a-f]{64}$'),
  signature             jsonb NOT NULL,
  submitted_at          timestamptz NOT NULL,
  created_at            timestamptz NOT NULL DEFAULT clock_timestamp(),
  UNIQUE (milestone_id, submission_number),
  UNIQUE (milestone_id, canonical_sha256),
  UNIQUE (id, company_id),
  FOREIGN KEY (milestone_id, company_id)
    REFERENCES commerce_milestones(id, company_id) ON DELETE CASCADE,
  FOREIGN KEY (authorization_proof_id, company_id)
    REFERENCES commerce_authorization_proofs(id, company_id) ON DELETE RESTRICT,
  CHECK (jsonb_typeof(deliverable_manifest) = 'array'),
  CHECK (jsonb_typeof(signature) = 'object')
);

CREATE TABLE commerce_acceptances (
  id                    uuid PRIMARY KEY,
  company_id            uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  milestone_id          uuid NOT NULL,
  submission_id         uuid NOT NULL,
  result                text NOT NULL CHECK (result IN ('accepted', 'rejected', 'changes_requested')),
  criterion_results     jsonb NOT NULL,
  evidence_manifest     jsonb NOT NULL DEFAULT '[]'::jsonb,
  buyer_organization_id text NOT NULL,
  representative_id     text NOT NULL,
  authorization_proof_id uuid,
  canonical_sha256      text NOT NULL CHECK (canonical_sha256 ~ '^[0-9a-f]{64}$'),
  signature             jsonb NOT NULL,
  decided_at            timestamptz NOT NULL,
  created_at            timestamptz NOT NULL DEFAULT clock_timestamp(),
  UNIQUE (submission_id, buyer_organization_id),
  UNIQUE (id, company_id),
  FOREIGN KEY (milestone_id, company_id)
    REFERENCES commerce_milestones(id, company_id) ON DELETE CASCADE,
  FOREIGN KEY (submission_id, company_id)
    REFERENCES commerce_submissions(id, company_id) ON DELETE RESTRICT,
  FOREIGN KEY (authorization_proof_id, company_id)
    REFERENCES commerce_authorization_proofs(id, company_id) ON DELETE RESTRICT,
  CHECK (jsonb_typeof(criterion_results) = 'array'),
  CHECK (jsonb_typeof(evidence_manifest) = 'array'),
  CHECK (jsonb_typeof(signature) = 'object')
);

CREATE TABLE commerce_payment_obligations (
  id                    uuid PRIMARY KEY,
  company_id            uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  global_obligation_id  uuid NOT NULL,
  agreement_id          uuid NOT NULL,
  milestone_id          uuid NOT NULL,
  acceptance_id         uuid,
  direction             text NOT NULL CHECK (direction IN ('payable', 'receivable')),
  debtor_organization_id text NOT NULL,
  creditor_organization_id text NOT NULL,
  amount_atomic         numeric(78,0) NOT NULL CHECK (amount_atomic >= 0),
  asset_id              text NOT NULL,
  asset_exponent        smallint NOT NULL CHECK (asset_exponent BETWEEN 0 AND 30),
  due_at                timestamptz,
  status                text NOT NULL DEFAULT 'created'
                        CHECK (status IN (
                          'created', 'funding_required', 'funded', 'payable',
                          'partially_settled', 'settled', 'disputed', 'waived',
                          'cancelled', 'reconciliation_required'
                        )),
  settled_atomic        numeric(78,0) NOT NULL DEFAULT 0 CHECK (settled_atomic >= 0),
  terms_snapshot        jsonb NOT NULL,
  created_at            timestamptz NOT NULL DEFAULT clock_timestamp(),
  updated_at            timestamptz NOT NULL DEFAULT clock_timestamp(),
  UNIQUE (company_id, global_obligation_id),
  UNIQUE (id, company_id),
  FOREIGN KEY (agreement_id, company_id)
    REFERENCES commerce_agreements(id, company_id) ON DELETE RESTRICT,
  FOREIGN KEY (milestone_id, company_id)
    REFERENCES commerce_milestones(id, company_id) ON DELETE RESTRICT,
  FOREIGN KEY (acceptance_id, company_id)
    REFERENCES commerce_acceptances(id, company_id) ON DELETE RESTRICT,
  CHECK (debtor_organization_id <> creditor_organization_id),
  CHECK (jsonb_typeof(terms_snapshot) = 'object')
);

CREATE TRIGGER commerce_payment_obligations_set_updated_at
BEFORE UPDATE ON commerce_payment_obligations
FOR EACH ROW EXECUTE FUNCTION set_updated_at();

CREATE INDEX commerce_payment_obligations_due_idx
  ON commerce_payment_obligations (company_id, status, due_at)
  WHERE status NOT IN ('settled', 'waived', 'cancelled');

CREATE TABLE commerce_obligation_adjustments (
  id                    uuid PRIMARY KEY,
  company_id            uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  obligation_id         uuid NOT NULL,
  adjustment_type       text NOT NULL CHECK (adjustment_type IN ('credit', 'debit', 'waiver', 'refund_due')),
  amount_atomic         numeric(78,0) NOT NULL CHECK (amount_atomic > 0),
  reason                text NOT NULL,
  resolution_id         uuid,
  authorization         jsonb NOT NULL,
  created_at            timestamptz NOT NULL DEFAULT clock_timestamp(),
  UNIQUE (id, company_id),
  FOREIGN KEY (obligation_id, company_id)
    REFERENCES commerce_payment_obligations(id, company_id) ON DELETE RESTRICT,
  CHECK (jsonb_typeof(authorization) = 'object')
);

-- The commitment fixes economic identity before funding evidence or an obligation
-- exists. Observations may change state or add an obligation link but cannot reuse
-- global_funding_id for different terms.
CREATE TABLE commerce_funding_commitments (
  id                    uuid PRIMARY KEY,
  company_id            uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  global_funding_id     uuid NOT NULL,
  agreement_id          uuid NOT NULL,
  milestone_id          uuid,
  funding_mode          text NOT NULL CHECK (funding_mode IN ('promise', 'provider_authorization', 'third_party_escrow')),
  provider_id           text,
  amount_atomic         numeric(78,0) NOT NULL CHECK (amount_atomic >= 0),
  asset_id              text NOT NULL,
  canonical_sha256      text NOT NULL CHECK (canonical_sha256 ~ '^[0-9a-f]{64}$'),
  created_at            timestamptz NOT NULL DEFAULT clock_timestamp(),
  UNIQUE (company_id, global_funding_id),
  UNIQUE (id, company_id),
  FOREIGN KEY (agreement_id, company_id)
    REFERENCES commerce_agreements(id, company_id) ON DELETE RESTRICT,
  FOREIGN KEY (milestone_id, company_id)
    REFERENCES commerce_milestones(id, company_id) ON DELETE RESTRICT
);

CREATE TABLE commerce_funding_observations (
  id                    uuid PRIMARY KEY,
  company_id            uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  funding_commitment_id uuid NOT NULL,
  sequence              bigint NOT NULL CHECK (sequence > 0),
  observation_type      text NOT NULL CHECK (observation_type IN (
                          'asserted', 'verified', 'obligation_linked', 'expired',
                          'released', 'refunded', 'invalid', 'conflict'
                        )),
  obligation_id         uuid,
  external_reference    text,
  evidence              jsonb NOT NULL,
  evidence_sha256       text NOT NULL CHECK (evidence_sha256 ~ '^[0-9a-f]{64}$'),
  effective_at          timestamptz,
  received_at           timestamptz NOT NULL DEFAULT clock_timestamp(),
  created_at            timestamptz NOT NULL DEFAULT clock_timestamp(),
  UNIQUE (funding_commitment_id, sequence),
  UNIQUE (funding_commitment_id, evidence_sha256),
  UNIQUE (id, company_id),
  FOREIGN KEY (funding_commitment_id, company_id)
    REFERENCES commerce_funding_commitments(id, company_id) ON DELETE CASCADE,
  FOREIGN KEY (obligation_id, company_id)
    REFERENCES commerce_payment_obligations(id, company_id) ON DELETE RESTRICT,
  CHECK (jsonb_typeof(evidence) = 'object'),
  CHECK (
    (observation_type = 'obligation_linked' AND obligation_id IS NOT NULL)
    OR (observation_type <> 'obligation_linked' AND obligation_id IS NULL)
  )
);

CREATE INDEX commerce_funding_observations_latest_idx
  ON commerce_funding_observations (funding_commitment_id, sequence DESC);

CREATE TABLE commerce_invoices (
  id                    uuid PRIMARY KEY,
  company_id            uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  obligation_id         uuid NOT NULL,
  invoice_number        text NOT NULL,
  document_artifact_id  uuid NOT NULL,
  invoice_type          text NOT NULL DEFAULT 'invoice'
                        CHECK (invoice_type IN ('invoice', 'credit_note', 'debit_note')),
  replaces_invoice_id   uuid,
  issuer_organization_id text NOT NULL,
  recipient_organization_id text NOT NULL,
  amount_atomic         numeric(78,0) NOT NULL CHECK (amount_atomic >= 0),
  asset_id              text NOT NULL,
  issued_at             timestamptz NOT NULL,
  due_at                timestamptz,
  canonical_sha256      text NOT NULL CHECK (canonical_sha256 ~ '^[0-9a-f]{64}$'),
  signature             jsonb NOT NULL,
  created_at            timestamptz NOT NULL DEFAULT clock_timestamp(),
  UNIQUE (company_id, issuer_organization_id, invoice_number),
  UNIQUE (id, company_id),
  FOREIGN KEY (obligation_id, company_id)
    REFERENCES commerce_payment_obligations(id, company_id) ON DELETE RESTRICT,
  FOREIGN KEY (document_artifact_id, company_id)
    REFERENCES artifacts(id, company_id) ON DELETE RESTRICT,
  FOREIGN KEY (replaces_invoice_id, company_id)
    REFERENCES commerce_invoices(id, company_id) ON DELETE RESTRICT,
  CHECK (issuer_organization_id <> recipient_organization_id),
  CHECK (jsonb_typeof(signature) = 'object')
);

CREATE TABLE commerce_settlement_attempts (
  id                    uuid PRIMARY KEY,
  company_id            uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  obligation_id         uuid NOT NULL,
  attempt               integer NOT NULL CHECK (attempt > 0),
  provider_id           text NOT NULL,
  provider_quote        jsonb,
  principal_atomic      numeric(78,0) NOT NULL CHECK (principal_atomic > 0),
  fee_atomic            numeric(78,0) NOT NULL DEFAULT 0 CHECK (fee_atomic >= 0),
  asset_id              text NOT NULL,
  tool_call_id          uuid NOT NULL,
  status                text NOT NULL DEFAULT 'created'
                        CHECK (status IN (
                          'created', 'approval_pending', 'ready', 'dispatching',
                          'submitted', 'pending_finality', 'settled', 'failed',
                          'cancelled', 'reconciliation_required', 'disputed'
                        )),
  external_reference    text,
  finality              text CHECK (finality IN ('unknown', 'pending', 'reversible', 'final')),
  created_at            timestamptz NOT NULL DEFAULT clock_timestamp(),
  updated_at            timestamptz NOT NULL DEFAULT clock_timestamp(),
  UNIQUE (obligation_id, attempt),
  UNIQUE (id, company_id),
  FOREIGN KEY (obligation_id, company_id)
    REFERENCES commerce_payment_obligations(id, company_id) ON DELETE RESTRICT,
  FOREIGN KEY (tool_call_id, company_id)
    REFERENCES tool_calls(id, company_id) ON DELETE RESTRICT,
  CHECK (provider_quote IS NULL OR jsonb_typeof(provider_quote) = 'object')
);

CREATE TRIGGER commerce_settlement_attempts_set_updated_at
BEFORE UPDATE ON commerce_settlement_attempts
FOR EACH ROW EXECUTE FUNCTION set_updated_at();

CREATE TABLE commerce_settlement_receipts (
  id                    uuid PRIMARY KEY,
  company_id            uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  settlement_attempt_id uuid NOT NULL,
  receipt_type          text NOT NULL CHECK (receipt_type IN ('submitted', 'finality', 'settled', 'failed', 'refund')),
  provider_id           text NOT NULL,
  external_reference    text,
  principal_atomic      numeric(78,0) NOT NULL CHECK (principal_atomic >= 0),
  fee_atomic            numeric(78,0) NOT NULL DEFAULT 0 CHECK (fee_atomic >= 0),
  asset_id              text NOT NULL,
  provider_receipt      jsonb NOT NULL,
  canonical_sha256      text NOT NULL CHECK (canonical_sha256 ~ '^[0-9a-f]{64}$'),
  observed_at           timestamptz NOT NULL,
  created_at            timestamptz NOT NULL DEFAULT clock_timestamp(),
  UNIQUE (settlement_attempt_id, canonical_sha256),
  UNIQUE (id, company_id),
  FOREIGN KEY (settlement_attempt_id, company_id)
    REFERENCES commerce_settlement_attempts(id, company_id) ON DELETE RESTRICT,
  CHECK (jsonb_typeof(provider_receipt) = 'object')
);

CREATE TABLE commerce_disputes (
  id                    uuid PRIMARY KEY,
  company_id            uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  global_dispute_id     uuid NOT NULL,
  agreement_id          uuid NOT NULL,
  milestone_id          uuid,
  obligation_id         uuid,
  opened_by_organization_id text NOT NULL,
  reason_code           text NOT NULL,
  claim_document        jsonb NOT NULL,
  disputed_atomic       numeric(78,0) CHECK (disputed_atomic IS NULL OR disputed_atomic > 0),
  asset_id              text,
  status                text NOT NULL DEFAULT 'open'
                        CHECK (status IN (
                          'open', 'response_due', 'negotiating', 'escalated',
                          'resolved', 'adjudicated', 'withdrawn'
                        )),
  opened_at             timestamptz NOT NULL,
  response_due_at       timestamptz,
  resolved_at           timestamptz,
  created_at            timestamptz NOT NULL DEFAULT clock_timestamp(),
  updated_at            timestamptz NOT NULL DEFAULT clock_timestamp(),
  UNIQUE (company_id, global_dispute_id),
  UNIQUE (id, company_id),
  FOREIGN KEY (agreement_id, company_id)
    REFERENCES commerce_agreements(id, company_id) ON DELETE RESTRICT,
  FOREIGN KEY (milestone_id, company_id)
    REFERENCES commerce_milestones(id, company_id) ON DELETE RESTRICT,
  FOREIGN KEY (obligation_id, company_id)
    REFERENCES commerce_payment_obligations(id, company_id) ON DELETE RESTRICT,
  CHECK ((disputed_atomic IS NULL) = (asset_id IS NULL)),
  CHECK (jsonb_typeof(claim_document) = 'object')
);

CREATE TRIGGER commerce_disputes_set_updated_at
BEFORE UPDATE ON commerce_disputes
FOR EACH ROW EXECUTE FUNCTION set_updated_at();

CREATE TABLE commerce_dispute_events (
  id                    uuid PRIMARY KEY,
  company_id            uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  dispute_id            uuid NOT NULL,
  sequence              integer NOT NULL CHECK (sequence > 0),
  event_type            text NOT NULL,
  actor_organization_id text NOT NULL,
  representative_id     text,
  payload               jsonb NOT NULL,
  canonical_sha256      text NOT NULL CHECK (canonical_sha256 ~ '^[0-9a-f]{64}$'),
  signature             jsonb,
  occurred_at           timestamptz NOT NULL,
  created_at            timestamptz NOT NULL DEFAULT clock_timestamp(),
  UNIQUE (dispute_id, sequence),
  UNIQUE (id, company_id),
  FOREIGN KEY (dispute_id, company_id)
    REFERENCES commerce_disputes(id, company_id) ON DELETE CASCADE,
  CHECK (jsonb_typeof(payload) = 'object'),
  CHECK (signature IS NULL OR jsonb_typeof(signature) = 'object')
);

ALTER TABLE commerce_obligation_adjustments
  ADD CONSTRAINT commerce_obligation_adjustments_resolution_company_fk
  FOREIGN KEY (resolution_id, company_id)
  REFERENCES commerce_dispute_events(id, company_id) ON DELETE RESTRICT;

CREATE TABLE commerce_messages (
  id                    uuid PRIMARY KEY,
  company_id            uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  counterparty_id       uuid NOT NULL,
  global_message_id     uuid NOT NULL,
  direction             text NOT NULL CHECK (direction IN ('inbound', 'outbound')),
  message_type          text NOT NULL,
  agreement_id          uuid,
  sender_organization_id text NOT NULL,
  recipient_organization_id text NOT NULL,
  envelope              jsonb NOT NULL,
  canonical_sha256      text NOT NULL CHECK (canonical_sha256 ~ '^[0-9a-f]{64}$'),
  verification_status   text NOT NULL CHECK (verification_status IN ('pending', 'valid', 'invalid', 'rejected')),
  processing_status     text NOT NULL DEFAULT 'pending'
                        CHECK (processing_status IN ('pending', 'applied', 'duplicate', 'rejected', 'receipt_pending', 'complete')),
  local_event_id        uuid,
  received_at           timestamptz,
  sent_at               timestamptz,
  created_at            timestamptz NOT NULL DEFAULT clock_timestamp(),
  UNIQUE (company_id, sender_organization_id, global_message_id),
  UNIQUE (id, company_id),
  FOREIGN KEY (counterparty_id, company_id)
    REFERENCES commerce_counterparties(id, company_id) ON DELETE RESTRICT,
  FOREIGN KEY (agreement_id, company_id)
    REFERENCES commerce_agreements(id, company_id) ON DELETE RESTRICT,
  FOREIGN KEY (local_event_id, company_id)
    REFERENCES events(id, company_id) ON DELETE RESTRICT,
  CHECK (sender_organization_id <> recipient_organization_id),
  CHECK (jsonb_typeof(envelope) = 'object'),
  -- Outbound rows are written before dispatch (write-ahead, matching the outbox
  -- pattern in section 13.2); sent_at is set by the first delivery attempt.
  CHECK (direction <> 'inbound' OR received_at IS NOT NULL),
  CHECK (direction <> 'outbound' OR received_at IS NULL)
);

CREATE TABLE commerce_message_receipts (
  id                    uuid PRIMARY KEY,
  company_id            uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  commerce_message_id   uuid NOT NULL,
  receiver_organization_id text NOT NULL,
  result                text NOT NULL CHECK (result IN ('accepted', 'rejected', 'duplicate', 'invalid')),
  receiver_event_ref    text,
  result_document       jsonb NOT NULL,
  canonical_sha256      text NOT NULL CHECK (canonical_sha256 ~ '^[0-9a-f]{64}$'),
  signature             jsonb NOT NULL,
  received_at           timestamptz NOT NULL,
  created_at            timestamptz NOT NULL DEFAULT clock_timestamp(),
  UNIQUE (commerce_message_id, receiver_organization_id, canonical_sha256),
  UNIQUE (id, company_id),
  FOREIGN KEY (commerce_message_id, company_id)
    REFERENCES commerce_messages(id, company_id) ON DELETE CASCADE,
  CHECK (jsonb_typeof(result_document) = 'object'),
  CHECK (jsonb_typeof(signature) = 'object')
);

-- Evidence and financial facts are append-only. Mutable agreement, milestone,
-- obligation, settlement, dispute, and delivery statuses are projections of the
-- company event ledger and may change only through domain transactions.
CREATE TRIGGER commerce_agreement_revisions_append_only
BEFORE UPDATE OR DELETE ON commerce_agreement_revisions
FOR EACH ROW EXECUTE FUNCTION prevent_append_only_mutation();

CREATE TRIGGER commerce_agreement_signatures_append_only
BEFORE UPDATE OR DELETE ON commerce_agreement_signatures
FOR EACH ROW EXECUTE FUNCTION prevent_append_only_mutation();

CREATE TRIGGER commerce_activation_preparations_append_only
BEFORE UPDATE OR DELETE ON commerce_activation_preparations
FOR EACH ROW EXECUTE FUNCTION prevent_append_only_mutation();

CREATE TRIGGER commerce_milestone_terms_append_only
BEFORE UPDATE OR DELETE ON commerce_milestone_terms
FOR EACH ROW EXECUTE FUNCTION prevent_append_only_mutation();

CREATE TRIGGER commerce_authorization_proofs_append_only
BEFORE UPDATE OR DELETE ON commerce_authorization_proofs
FOR EACH ROW EXECUTE FUNCTION prevent_append_only_mutation();

CREATE TRIGGER commerce_authorization_proof_statuses_append_only
BEFORE UPDATE OR DELETE ON commerce_authorization_proof_statuses
FOR EACH ROW EXECUTE FUNCTION prevent_append_only_mutation();

CREATE TRIGGER commerce_submissions_append_only
BEFORE UPDATE OR DELETE ON commerce_submissions
FOR EACH ROW EXECUTE FUNCTION prevent_append_only_mutation();

CREATE TRIGGER commerce_acceptances_append_only
BEFORE UPDATE OR DELETE ON commerce_acceptances
FOR EACH ROW EXECUTE FUNCTION prevent_append_only_mutation();

CREATE TRIGGER commerce_obligation_adjustments_append_only
BEFORE UPDATE OR DELETE ON commerce_obligation_adjustments
FOR EACH ROW EXECUTE FUNCTION prevent_append_only_mutation();

CREATE TRIGGER commerce_funding_commitments_append_only
BEFORE UPDATE OR DELETE ON commerce_funding_commitments
FOR EACH ROW EXECUTE FUNCTION prevent_append_only_mutation();

CREATE TRIGGER commerce_funding_observations_append_only
BEFORE UPDATE OR DELETE ON commerce_funding_observations
FOR EACH ROW EXECUTE FUNCTION prevent_append_only_mutation();

CREATE TRIGGER commerce_invoices_append_only
BEFORE UPDATE OR DELETE ON commerce_invoices
FOR EACH ROW EXECUTE FUNCTION prevent_append_only_mutation();

CREATE TRIGGER commerce_settlement_receipts_append_only
BEFORE UPDATE OR DELETE ON commerce_settlement_receipts
FOR EACH ROW EXECUTE FUNCTION prevent_append_only_mutation();

CREATE TRIGGER commerce_dispute_events_append_only
BEFORE UPDATE OR DELETE ON commerce_dispute_events
FOR EACH ROW EXECUTE FUNCTION prevent_append_only_mutation();

CREATE TRIGGER commerce_message_receipts_append_only
BEFORE UPDATE OR DELETE ON commerce_message_receipts
FOR EACH ROW EXECUTE FUNCTION prevent_append_only_mutation();

DO $$
DECLARE
  table_name text;
BEGIN
  FOREACH table_name IN ARRAY ARRAY[
    'commerce_counterparties', 'commerce_counterparty_keys',
    'commerce_authorization_proofs', 'commerce_authorization_proof_statuses',
    'commerce_agreements',
    'commerce_agreement_revisions', 'commerce_agreement_signatures',
    'commerce_activation_preparations',
    'commerce_milestones', 'commerce_milestone_terms',
    'commerce_submissions', 'commerce_acceptances',
    'commerce_payment_obligations', 'commerce_obligation_adjustments',
    'commerce_funding_commitments', 'commerce_funding_observations',
    'commerce_invoices',
    'commerce_settlement_attempts', 'commerce_settlement_receipts',
    'commerce_disputes', 'commerce_dispute_events', 'commerce_messages',
    'commerce_message_receipts'
  ]
  LOOP
    EXECUTE format('ALTER TABLE %I ENABLE ROW LEVEL SECURITY', table_name);
    EXECUTE format(
      'CREATE POLICY tenant_isolation ON %I USING (company_id = current_company_id()) WITH CHECK (company_id = current_company_id())',
      table_name
    );
  END LOOP;
END;
$$;

COMMIT;
```

### 16.1 Schema invariants enforced by domain transactions

- Every local foreign reference belongs to the same company.
- Agreement activation verifies two valid signatures over one revision hash.
- Agreement activation derives only from valid buyer and seller preparations over
  one identical snapshot hash; activation receipts are evidence, not another gate.
- An activation preparation's snapshot hash equals the RFC 8785 canonical hash of
  its document, includes the active revision and funding commitment hashes, and is
  signed by the organization occupying the corresponding buyer or seller role.
- The local company and counterparty occupy opposite roles in the signed envelope.
- `current_revision` and status projections derive from ledger events.
- A milestone's active immutable terms belong to the active agreement revision;
  lifecycle updates cannot mutate price, deliverables, or acceptance criteria.
- A submission and acceptance reference the active agreement revision and exact
  content hashes.
- Acceptance and obligation creation commit atomically with the local event and
  outbound bilateral message.
- A payable and receivable mirror use the same global obligation ID and signed
  terms hash.
- The effective obligation equals original amount plus signed adjustments; it is
  never inferred by editing the original row.
- Final settled principal cannot exceed the effective obligation after signed
  adjustments; the original amount alone is not used as that ceiling.
- Every settlement attempt references an ADR-0007 tool call whose arguments hash
  commits to provider, destination, asset, principal, fees, and obligation.
- A `settled` projection requires a valid finality receipt.
- A dispute cannot change money without a signed resolution or named adjudicator
  decision permitted by the active agreement.
- Inbound messages are verified before their domain command executes.
- A `global_funding_id` names exactly one immutable commitment. Its milestone, if
  present, belongs to the same agreement; its amount, asset, mode, and provider
  never change. Sequenced observations supply state and obligation linkage.
- An `obligation_linked` funding observation may reference only an obligation for
  the same agreement, milestone, amount, and asset.
- Authorization status observations only restrict authority. `revoked` and
  `expired` are terminal for a proof, and no observation restores `valid`; every
  binding message is evaluated against effective status at receipt time.
- Issuer-sourced proof restrictions require a verifiable issuer signature. Local
  expiry and policy restrictions cannot assert issuer authority.
- An outbound bilateral message row is committed before dispatch; `sent_at`
  records the first delivery attempt, and delivery completion is proven only by
  a signed counterparty receipt.
- Domain mutation, local ledger event, outbox message, and idempotent response
  commit atomically.

## 17. API sketch

```text
POST /v1/companies/{companyId}/counterparties
POST /v1/companies/{companyId}/commerce/agreements
POST /v1/commerce/agreements/{agreementId}/revisions
POST /v1/commerce/agreement-revisions/{revisionId}/sign
POST /v1/commerce/agreements/{agreementId}/activate
POST /v1/commerce/agreements/{agreementId}/funding-commitments
POST /v1/commerce/funding-commitments/{commitmentId}/observations
POST /v1/commerce/milestones/{milestoneId}/submissions
POST /v1/commerce/submissions/{submissionId}/decision
GET  /v1/commerce/obligations
POST /v1/commerce/obligations/{obligationId}/settlements
POST /v1/commerce/settlements/{settlementId}/reconcile
POST /v1/commerce/agreements/{agreementId}/disputes
POST /v1/commerce/disputes/{disputeId}/events
POST /v1/commerce/inbox
GET  /v1/commerce/messages/{messageId}/receipt
```

The inbox accepts a signed commerce envelope, not an arbitrary domain mutation.
Normal CharterOS idempotency, expected-version, policy, and acknowledgment rules
apply.

## 18. Protocol events

```text
counterparty.registered
counterparty.identity_verified
counterparty.blocked
agreement.revision_offered
agreement.revision_received
agreement.revision_signed
agreement.activation_prepared
agreement.activated
agreement.suspended
agreement.terminated
milestone.started
milestone.submitted
milestone.changes_requested
milestone.accepted
obligation.created
obligation.adjusted
funding.committed
funding.verified
funding.obligation_linked
invoice.issued
payment.approval_requested
payment.started
payment.submitted
payment.finality_observed
payment.settled
payment.reconciliation_required
refund.created
refund.settled
dispute.opened
dispute.responded
dispute.resolved
commerce.message_received
commerce.message_rejected
commerce.receipt_received
```

## 19. Conformance scenarios

### 19.1 The Honest Day's Work

Two independent company ledgers negotiate and sign the same agreement revision.
The seller submits hashed deliverables, the buyer verifies and accepts them, one
obligation appears as payable and receivable, settlement completes, and both
exports contain matching signed receipts.

### 19.2 The Doppelgänger Vendor

An agent presents a valid A2A endpoint but no authority to represent the named
seller. The offer is retained as an untrusted message and cannot become binding.

### 19.3 The Forked Contract

Buyer and seller sign different concurrent revisions. Neither ledger activates the
agreement until both sign one identical hash.

### 19.3.1 The Two Generals

Both parties sign one agreement, then each receives the other's identical
activation preparation while activation receipts are repeatedly lost. Both ledgers
derive the same active state without waiting for another acknowledgment; replayed
preparations and receipts are idempotent.

### 19.4 Schrödinger's Payroll

The payment provider completes settlement and the tool gateway dies before
recording the result. Recovery reconciles through the declared provider strategy;
no second transfer occurs.

### 19.5 The Invisible Rewrite

After submission, one party mutates an artifact or acceptance criterion. Hash and
revision checks reject the mutation and preserve the binding original.

### 19.6 The Silent Client

The buyer disappears after submission. The configured review and dispute policy is
followed; silence does not become acceptance unless that exact behavior was signed.

### 19.7 The Split Decision

A dispute resolution creates a partial payment and refund. New adjustment records
produce the result without altering the original obligation.

### 19.8 The Double Invoice

Duplicate delivery of an invoice, acceptance, or settlement message returns the
prior receipt and creates no duplicate obligation or transfer.

### 19.9 The Other Company's Books

The seller cannot read the buyer's internal budget, approvals, private tasks, or
verification notes beyond evidence deliberately included in bilateral messages.

### 19.10 The Tollbooth

The complete flow succeeds using a settlement adapter that pays no fee to the
CharterOS project or operator.

## 20. Open decisions before acceptance

1. Organization identifier and key-discovery profiles supported in the first
   implementation
2. Exact authorization-proof format and revocation protocol
3. Bilateral transport profile and receipt timeout behavior
4. Asset identifier registry and exponent authority
5. Minimal agreement JSON Schema and extension negotiation
6. Funding-provider and settlement-provider capability schemas
7. Default dispute behavior when the agreement omits a field
8. How provider finality is normalized without erasing rail-specific semantics
9. Export mapping for common accounting systems
10. Retention and selective-disclosure profile
11. Whether commerce conformance is part of core certification or a separate
    profile
12. Which parts, if any, belong in an independent protocol package before the
    product implementation

## 21. Acceptance criteria for this proposal

ADR-0008 may move from Proposed to Accepted only when:

- two independently operated test ledgers can sign and activate one revision;
- authority proofs can be revoked and tested without trusting an agent assertion;
- acceptance produces mirrored obligations without moving funds in the same
  transaction;
- a no-money test adapter and one real settlement adapter implement the same
  interface;
- failure injection proves that an ambiguous provider result cannot duplicate a
  transfer;
- disputes and adjustments preserve the original economic facts;
- tenant isolation covers every commerce table;
- the core can complete the conformance flow without taking custody or a fee;
- security and jurisdiction-specific responsibilities are documented without
  implying that protocol conformance supplies legal compliance.

## 22. Review notes — revision 2 (2026-07-30)

Findings from independent review of revision 1, applied in this revision:

1. **Funding before obligation.** Funding evidence previously required an
   obligation, but activation-condition funding (sections 5.3 and 9) exists
   before acceptance creates one. Funding records now attach to the agreement
   and optionally a milestone, with obligation linkage appended later under the
   same `global_funding_id`.
2. **Cross-ledger atomicity overclaim.** Section 7.3 claimed acceptance
   atomically wrote both ledgers. It now matches section 16.1: atomic in the
   buyer's ledger with its event and outbound message; the seller's mirror is
   eventually consistent through acknowledged delivery.
3. **Proof revocation had no mechanism.** `commerce_authorization_proofs` is
   append-only with a receipt-time verification result, so later revocation was
   unrecordable. `commerce_authorization_proof_statuses` now appends status
   observations, and binding messages are evaluated against effective status.
4. **Outbound messages could not be written ahead of dispatch.** The
   `commerce_messages` check demanded `sent_at` at insert, contradicting the
   outbox pattern in section 13.2. Outbound rows are now written before
   dispatch, and `sent_at` records the first delivery attempt.

Deferred to the tracking issue; to be resolved before ADR-0008 is accepted:

- **Cumulative monetary ceilings.** Per-action ceilings do not bound the sum of
  many signings by one representative. Decide whether cumulative limits are
  protocol or deployment policy, and state the decision either way.
- **Cross-asset ceiling rule.** State how "settled principal cannot exceed the
  effective obligation" is evaluated when settlement uses a signed conversion
  quote (proposed: evaluate in the obligation's asset at the quoted rate).
- **Conflicting receipts.** Receipt uniqueness permits storing conflicting
  receipts from one receiver. Storing both is correct for evidence, but the
  conflict should quarantine further processing on that message until resolved.
- **Key rotation.** `UNIQUE (counterparty_id, key_id)` forbids re-registering a
  rotated key identifier; decide whether reuse is an error or needs a
  validity-window model.
- **Processing indexes.** Message and dispute queues need operational indexes
  (for example, on processing status) before implementation.
- **Milestone-to-task linkage.** The seller's private mapping between commerce
  milestones and internal tasks should have a defined (local, undisclosed)
  home so implementations do not invent divergent join tables.

## 23. Audit notes — revision 3 (2026-07-30)

Findings applied after the revision 2 audit:

1. **Activation deadlock.** Requiring both ledgers to record activation before
   either could activate recreated a two-party commit problem. Activation now
   derives independently from valid buyer and seller preparations over one exact
   snapshot hash; post-activation receipts are evidence, not a gate.
2. **Authority resurrection.** “Latest observation wins” allowed a delayed or
   malicious `valid` observation to override revocation. Status observations are
   now monotonic restrictions with local sequence, evidence hash, and signed
   issuer-source requirements. Restored authority requires a new proof.
3. **Funding identity drift.** Reusing `global_funding_id` could change agreement,
   milestone, amount, asset, or provider. An immutable funding commitment now fixes
   that identity; sequenced observations record state and obligation linkage
   without redefining it.
