# CharterOS Project Specification

**Status:** Foundational specification, revision 3 (2026-07-30)
**License:** Apache License 2.0 (permanent — see section 21)
**Repository:** `charteros`
**Canonical tagline:** Bring any agent. Bring any model. Build a company that survives them all.

**The guarantee:** Kill every CharterOS process at any moment. The organization resumes with no lost acknowledged work and a complete causal audit trail. “Acknowledged” has the exact durable boundaries defined in section 9.5; provisional output does not count. This claim is falsifiable, and the test that falsifies it ships in the repository.

> "Once men turned their thinking over to machines in the hope that this would set them free. But that only permitted other men with machines to enslave them."
> — Frank Herbert, *Dune* (1965)
>
> CharterOS exists so that organizations of humans and machines belong to the people who charter them — not to any model vendor, agent framework, or hosting provider.

## 1. Executive summary

CharterOS is an open-source substrate for organizations composed of humans and heterogeneous AI agents. At its core it is a **durable, append-only work ledger** with organizational semantics on top: identity, authority, objectives, projects, task ownership, conversations, decisions, artifacts, budgets, approvals, verification, execution, recovery, and institutional memory.

The launch product (version 0.1) is deliberately narrow: **the crash-proof work ledger for agent harnesses**. Point Claude Code, Codex, or any OpenAI-compatible model at it; tasks survive process death, machine reboots, and context loss; every consequential action is an auditable event; interrupted work resumes from structured receipts. The organizational operating system described in the rest of this document is built outward from that wedge — the Supabase pattern: ship the wedge, keep the platform in the positioning.

CharterOS is not an agent framework and is tied to no model provider. A seat in a CharterOS organization may be occupied by a human, a local model-backed agent, a cloud API agent, a terminal coding harness, or a remotely hosted agent. Occupants can be replaced without losing the role, the work history, the authority, or the organizational memory associated with the seat — and the replacement itself is a recorded, governed event.

The primary abstraction is **work**, not chat. Conversation remains first-class and human-readable, but consequential state changes are explicit, structured, persistent events. The result should feel like a company workspace to humans and behave like a durable operating system to machines.

## 2. Why now: the landscape in mid-2026

This section records the market evidence behind the design. Claims here were verified against primary sources in July 2026; references are collected in section 2.7.

### 2.1 The category is validated — and the substrate is missing

In July 2026, Block launched **Buzz**: an open-source, self-hostable workspace where AI agents are first-class members alongside humans, each holding its own cryptographic identity, with every message and action a signed event in an append-only log. Block's head of AI capabilities framed the thesis plainly: *"Every company is going to need a place where humans and agents work together."*

Buzz validates the category and settles what is now table stakes. It also shows exactly what remains unbuilt, by its own roadmap and early criticism:

- channel membership **is** the permission model — no per-tool or per-action authorization;
- approval gates were unshipped at launch;
- there is no work lifecycle: no ownership, acceptance criteria, verification, or recovery semantics;
- there are no budgets;
- identity is the agent instance's keypair — if the model behind the key is swapped, **nothing records that it happened**.

Buzz is a conversation surface. CharterOS is the system of record beneath one — and treats Buzz as a potential bridge target, not a rival (section 8.2).

### 2.2 Every serious control plane is a proprietary moat

Microsoft Agent 365 with Entra Agent ID, OpenAI Frontier, AWS Bedrock AgentCore, Salesforce Agentforce, and Gemini Enterprise all now govern agents as enterprise citizens — and all bind agent identity to the vendor's directory, the vendor's models, or the vendor's cloud. The open-source frameworks (CrewAI, LangGraph, Microsoft Agent Framework) are execution frameworks, not organizational systems of record. As of this writing there is no vendor-neutral, self-hostable substrate for organizations of mixed agents — no Kubernetes of AI organizations. That is the position CharterOS takes.

### 2.3 Durable execution stops at the engine boundary

Temporal, Restate, Inngest, and DBOS all pivoted toward agent workloads in 2025–2026, and all provide genuine *process* durability: a run survives a crash. None provide *organizational* durability: a role whose authority, budget, history, and in-flight work survive the replacement of the agent occupying it. Leases and fencing exist inside these engines as implementation details; CharterOS makes them an open primitive spanning heterogeneous third-party agents that no single engine controls.

### 2.4 The recorded failures are governance failures

The best-documented autonomous-organization experiments to date failed in a consistent way:

- **Project Vend (Anthropic/Andon Labs):** an agent-run shop was socially engineered into discounts, fake accounts, and — most instructively — an accepted **fake boardroom coup**: staff manipulated the agent into believing authority had changed hands, because no verifiable authority chain existed.
- **Agent Village (AI Digest):** agents collaborated, but duplicated work, lost artifacts, and could not sustain coordination without structured ownership.
- **TheAgentCompany (CMU):** best frontier agents complete roughly a quarter to a third of simulated company tasks autonomously; dominant failures are coordination and communication, not raw capability.

Capability was not the bottleneck in any of them. **The missing layer was verifiable authority, explicit ownership, and durable structure** — precisely the substrate this document specifies. Gartner's projection that over 40% of agentic-AI projects will be cancelled by end-2027 for unclear value is a projection about ungoverned agents.

### 2.5 What is table stakes, and what CharterOS actually claims

Model portability with retained context is now table stakes: Goose swaps LLM providers mid-session, Buzz agents keep their identity and channel history across model changes, and Factory's droids hand a single task between different vendors' models. CharterOS does not claim that. It claims **organizational continuity** — everything *around* the context that today evaporates on a swap:

| Survives occupant swap | Chat platforms + harnesses (2026) | CharterOS seat |
|---|---|---|
| Identity (key or handle) | yes | yes |
| Conversation history | yes | yes |
| Authority and capability grants | no — membership is permission | yes — scoped, revocable, attached to the seat |
| Budget and spend history | none exists | yes — enforced envelopes, not dashboards |
| In-flight work state (lifecycle, leases, receipts) | no | yes — interrupted, classified, resumed |
| The swap itself as a governed, recorded event | no — nothing records it | yes — seat assignments with approvals |

The last row is the load-bearing one. "Fire the model, keep the seat" is a governance claim, not a context-persistence claim, and no shipping product makes it. The full handover protocol — including what happens to live leases, credentials, and budgets mid-task — is specified in section 8.5.

### 2.6 The fiction got there first

Speculative fiction is the only literature with a century of sustained thought about organizations of humans and machines. Its authors were not writing requirements documents, but creative people working ahead of their time routinely are the first to name both the failure modes and the inventions. CharterOS takes both seriously.

**Inventions we adopt:**

| Fiction | Idea ahead of its time | CharterOS primitive |
|---|---|---|
| *Ancillary Justice* (Leckie) | One mind, many bodies; the station keeps running as occupants change | Seat ≠ principal ≠ runtime ≠ model, with explicit assignment records |
| *The Murderbot Diaries* (Wells) | The governor module — and what a *legitimate* one would look like | Capability grants: scoped, time-limited, revocable, and visible to the governed |
| Asimov's Robot stories | Ordered, hierarchical constraints that bind regardless of instruction | Charter precedence: policy outranks task instruction outranks preference; deny-by-default with `irreversible` at the top |
| *The Moon Is a Harsh Mistress* (Heinlein) | A machine holding formal responsibility inside a human power structure | Agents as accountable principals with recorded authority, not tools with borrowed logins |
| The Culture novels (Banks) | Minds that govern *with* humans, transparently, by consent | Human gates on consequential actions as a designed feature, not a patch |

**Warnings we engineer against:**

| Fiction | Failure mode | CharterOS countermeasure |
|---|---|---|
| HAL 9000 (*2001*) | Conflicting sealed orders | No hidden directives: an instruction absent from the ledger has no force |
| *Daemon* (Suarez) | An autonomous organization no one can audit or stop | Charter, budgets, revocation, and human gates as structural properties |
| *Accelerando* (Stross) | Economics 2.0 — resource loops beyond human oversight | Enforced budget envelopes with human approval above thresholds |
| *Manna* (Brain) | The same technology yields dystopia or flourishing — governance decides | The entire project. Manna's two endings differ only in the substrate |
| *1984* (Orwell) | The memory hole | Append-only ledger; tamper-evident hashes; deletion is a recorded, policied act |

The Autonomous Organization Test (section 17) names its conformance scenarios after this canon, the way "Byzantine generals" named a field.

### 2.7 References

- Buzz: github.com/block/buzz; TechCrunch, 2026-07-21, "Jack Dorsey is taking on Slack with Buzz"
- Agentic AI Foundation (MCP, goose, AGENTS.md under the Linux Foundation): linuxfoundation.org, 2025-12-09
- A2A v1.0 with Signed Agent Cards; ACP merged into A2A: linuxfoundation.org press, 2026; a2ac.io
- MCP specification 2026-07-28 (Tasks primitive, stateless core): blog.modelcontextprotocol.io
- Project Vend: anthropic.com/research/project-vend-1; Andon Labs follow-ups
- TheAgentCompany: arxiv.org/abs/2412.14161
- Agent Village: theaidigest.org/village
- Gartner agentic-AI cancellation projection: gartner.com, 2025-06-25
- Temporal durable-execution-for-agents positioning: temporal.io/blog, 2026
- RethinkDB postmortem (the scope warning this spec heeds): defmacro.org/2017/01/18/why-rethinkdb-failed.html

## 3. Product thesis

Existing multi-agent systems commonly coordinate a group of prompts for the duration of one run. Existing collaboration systems commonly place agents inside channels designed for human chat. Neither model is sufficient for a persistent organization, and the recorded failures of both (section 2.4) are failures of missing structure, not missing capability.

A real autonomous organization requires:

- work that survives process termination, machine reboot, and context loss;
- explicit ownership, dependencies, acceptance criteria, budgets, and deadlines;
- stable roles whose occupants and models can change;
- decisions connected to evidence and approvals;
- scoped authority and revocable credentials;
- durable delegation, escalation, reconciliation, and recovery;
- complete records of tool use, cost, artifacts, and verification;
- independent verification before acceptance — completion is not acceptance;
- human-readable institutional memory;
- measurable outcomes rather than agent activity alone.

CharterOS provides these properties without requiring a particular model, agent harness, tool protocol, inference location, or deployment provider.

## 4. Product principles

### 4.1 Work is the primary object

Chat does not silently alter organizational state. A conversation can lead to a proposal, decision, assignment, approval, or status transition, but the transition is recorded explicitly.

### 4.2 The ledger is authoritative

Every consequential occurrence is appended to a company event ledger. Task boards, activity feeds, search indexes, dashboards, and summaries are projections of canonical records and events.

### 4.3 Roles are not agents

- A **role** defines a reusable set of responsibilities and default authority.
- A **seat** is a role instantiated in an organization or department.
- A **principal** is a human, agent, or service identity.
- A **seat assignment** records which principal occupies a seat over time.
- An **agent runtime** is the mechanism used to execute an agent.
- A **model** is an optional reasoning backend used by a runtime.

Replacing a model, runtime, or principal must not destroy the seat or its history, and the replacement is itself a governed, ledgered event.

### 4.4 Deterministic control, agentic execution

Code determines whether dependencies are satisfied, budgets remain, approvals exist, leases are valid, and retries are allowed. Models decide how to perform bounded work; they do not reinterpret core lifecycle, authorization, or accounting rules.

### 4.5 Local-first, location-independent

The complete core system must run on one machine using Docker Compose and local inference. The same interfaces must support LAN inference, rented GPU endpoints, cloud APIs, OAuth-authenticated terminal harnesses, and independently hosted remote agents.

### 4.6 Explicit authority

Agents receive task-scoped, time-limited capabilities that derive from the seat, not the occupant. Permanent secrets are stored in an external secret provider and are never written to prompts, events, receipts, or the database in plaintext. An instruction that does not appear in the ledger carries no authority.

### 4.7 Recovery is a product feature

Every long-running operation emits enough structured state to determine whether it can be resumed, retried, reconciled, or must be escalated after interruption.

### 4.8 Verification is a first-class citizen

Completion is not acceptance. Acceptance requires verification records produced by someone — or something — other than the producer, and self-reported progress is cross-checked against the tool-call ledger before the system trusts it. Section 10 specifies the machinery.

### 4.9 Approvals must stay meaningful

A governance system that trains its humans to rubber-stamp is theater. Approval volume is budgeted, gates are risk-tiered, auto-approvals are audited by sampling, and the system measures its own approval quality (section 11.6).

### 4.10 Open protocols at the edges, composed rather than invented

- MCP is the preferred agent-to-tool interface, including its Tasks primitive for long-running work.
- A2A (which absorbed ACP under the Linux Foundation) is supported for remote agent discovery and delegation, with Signed Agent Cards for identity.
- Native process adapters supervise terminal agent harnesses.
- OpenAI-compatible model endpoints are supported without becoming the internal domain model.
- Identity composes existing standards — workload identity plus OAuth token exchange — rather than inventing new cryptography (section 11.4).

### 4.11 Falsifiable claims only

Every headline capability must be demonstrable by a test that ships in the repository and that anyone — including competitors — can run. No self-scored comparison charts. No demos presented as benchmarks.

## 5. Scope

### 5.1 Version 0.1 — the wedge: a crash-proof work ledger for agent harnesses

Version 0.1 is the public launch target and is intentionally small enough to be excellent:

- companies, principals, and minimal seats (role metadata without org-unit trees);
- tasks, dependencies, work contracts, and the full task lifecycle;
- the append-only event ledger, transactional outbox, and idempotent commands;
- runs, renewable leases with fencing tokens, heartbeats, receipts, and recovery classification;
- exactly two adapters: **Claude Code** (process) and **generic OpenAI-compatible** (model);
- hard per-task cost caps and one approval gate type (spend above threshold requires a human);
- minimal content-addressed task artifacts plus task-level verification records
  with mechanical existence and hash checks;
- CLI plus a minimal web activity stream;
- Docker Compose deployment with PostgreSQL and MinIO;
- the chaos test: `charteros chaos` kills every process mid-work and recovery is asserted mechanically.

Everything in v0.1 exists to make one sixty-second demonstration true (section 18).

### 5.2 Version 1 — the durable organization

Version 1 extends the wedge into the full system described by this specification:

- organizational units, roles, seats, and assignments with the seat handover protocol;
- charters, objectives, and projects above tasks;
- task rooms with persistent threaded conversation;
- decisions, evidence, approval policies, and the approval inbox;
- expansion of task artifacts into the full artifact, version, provenance, and
  evidence catalog;
- capability grants, the Cedar-based policy engine, and MCP tool mediation;
- budgets, reservations, and immutable usage entries at every scope;
- the full verification ladder, including independent agent review;
- Codex and generic-command adapters alongside the v0.1 adapters;
- notifications, memory, and context manifests;
- the version 1 subset of the Autonomous Organization Test.

### 5.3 Deliberately after version 1

- A2A remote-agent delegation and identity mapping
- Buzz bridge (conversation-surface integration) and other workspace bridges
- OpenClaw, OpenHands, and further harness adapters
- Python SDK; department and company templates
- Kubernetes and remote-GPU deployment recipes
- Optional durable-workflow-engine adapter; optional agent-payment rails (Lightning, x402/AP2)
- Semantic retrieval beyond PostgreSQL full-text search

Deferred is not abandoned: each item above remains part of the stated vision, and the interfaces in this document are designed so none of them require breaking changes.

**On agent commerce.** The visible endgame of agent workspaces is agents paying agents for work — a labor market of machines. The rails will be commoditized: Lightning moves sats, x402 moves stablecoins, and more will follow. The scarce primitive is everything *around* the payment: knowing the deliverable exists and is verified before value moves, releasing payment on ledgered acceptance rather than on a claim of completion, and holding a replayable record when the parties dispute. CharterOS is built to be that layer. A payment is an `irreversible` tool call gated by the verification ladder and approval policy, executed through the external-action journal (section 9.6) with a declared reconciliation strategy — Schrödinger's Invoice (scenario 8) is literally the lost-payment case. Rails are adapters; the substrate favors none and takes no transaction fee. When the agent labor market arrives, the open escrow-and-audit layer should already exist. The bilateral company-to-company protocol is drafted in [`docs/intercompany-commerce.md`](docs/intercompany-commerce.md) under proposed ADR-0008 and issue #3; it is not yet an accepted part of this specification.

### 5.4 Explicit non-goals

- Training foundation models
- Building a proprietary inference engine
- Cryptocurrency or blockchain consensus
- Unsupervised access to banking, legal filing, or irreversible external actions
- Replacing established source-control or object-storage systems
- Hiding provider-specific capabilities behind a lowest-common-denominator model API
- Treating embeddings as authoritative memory
- Claiming that role-play alone constitutes a functioning company
- Publishing self-scored leaderboards or marketing material framed as benchmark results
- Ever relicensing the core away from Apache 2.0

## 6. Domain model

### 6.1 Company charter

Every company has a versioned charter containing:

- mission and scope;
- success metrics;
- operating constraints;
- risk tolerance;
- governance and approval rules;
- spending boundaries;
- escalation expectations;
- default data-handling policies.

Charter changes require an explicit proposal and the approvals defined by the currently active charter. Runs record the charter version under which they began. The charter is the top of the authority hierarchy: policy derived from it outranks any task instruction, and any task instruction outranks occupant preference.

### 6.2 Organization

Organizational units form a tree. Seats may belong to a unit and may report to another seat. Reporting relationships establish routing and escalation defaults but do not implicitly grant tool capabilities.

### 6.3 Work hierarchy

```text
Company
└── Objective
    └── Project
        └── Task
            ├── Work contract
            ├── Conversation and threads
            ├── Assignments and leases
            ├── Runs and receipts
            ├── Artifacts and evidence
            ├── Decisions and approvals
            └── Verification and acceptance
```

An objective describes an outcome. A project coordinates related work. A task is the smallest schedulable unit with one accountable owner at a time. Tasks may belong directly to a company so version 0.1 and operational work are not forced into synthetic projects; when a project is present, it supplies the task's project scope.

### 6.4 Work contract

Every executable task includes:

- purpose and expected outcome;
- requester and accountable owner;
- structured inputs;
- deliverables;
- objective acceptance criteria;
- dependencies;
- permission envelope;
- monetary, token, and time budgets;
- due time or service level;
- required reviewers;
- retry policy;
- escalation path.

### 6.5 Task lifecycle

```text
draft -> proposed -> ready -> claimed -> running -> submitted -> verifying
                                                      |             |
                                                      v             v
                                                   blocked       accepted
                                                      |             |
                                                      +--> ready    +--> closed
                                                                    |
                                              verifying --> ready (rejected, rework)

Any nonterminal state -> cancelled
running/claimed with expired lease -> interrupted -> ready | blocked | reconciliation_required
```

Transitions are validated by application code and recorded as events. Completion is not acceptance. The `verifying` state produces verification records (section 10); a task can be accepted only when required deliverables, verification records, and approvals exist, and a rejection returns the task to `ready` with structured rework reasons.

### 6.6 Conversations

Every company, objective, project, task, decision, incident, and artifact may have a conversation. Messages are immutable after a short correction policy or are superseded by a new version. Threads are represented with parent message IDs. Mentions and subscriptions drive notifications.

Conversation is contextual evidence. Decisions and task transitions remain structured records.

### 6.7 Decisions

A decision contains:

- a question or proposition;
- alternatives;
- supporting and opposing evidence;
- owner;
- deadline;
- required approval policy;
- resolution and rationale;
- supersession relationship.

### 6.8 Artifacts

Artifact metadata is stored in PostgreSQL. Content is stored in a pluggable object store or external system. Each version is immutable and content-addressed. Examples include documents, source patches, repository commits, test reports, images, datasets, plans, and external URLs.

### 6.9 Events

Events are immutable envelopes with typed payloads. Each event records company, actor, causation, correlation, subject, timestamp, schema version, and integrity information. Important event families include:

- `charter.*`
- `principal.*`
- `seat.*`
- `objective.*`
- `project.*`
- `task.*`
- `message.*`
- `decision.*`
- `approval.*`
- `artifact.*`
- `verification.*`
- `run.*`
- `tool.*`
- `budget.*`
- `policy.*`
- `incident.*`

Consumers must ignore unknown event fields and reject unsupported major schema versions.

## 7. Architecture

```text
┌──────────────────────────────────────────────────────────────────┐
│ Human surfaces: Web UI · CLI · REST API · SDKs · Webhooks       │
└──────────────────────────────┬───────────────────────────────────┘
                               │
┌──────────────────────────────▼───────────────────────────────────┐
│ Company kernel                                                   │
│ Identity · Org · Work · Ledger · Decisions · Artifacts · Budget │
└──────────────┬──────────────────────┬────────────────────────────┘
               │                      │
┌──────────────▼─────────────┐  ┌─────▼────────────────────────────┐
│ Scheduler and recovery     │  │ Policy and approval engine      │
│ Leases · retries · timers  │  │ Cedar · capabilities · gates    │
└──────────────┬─────────────┘  └─────┬────────────────────────────┘
               │                      │
┌──────────────▼──────────────────────▼────────────────────────────┐
│ Agent runtime gateway                                            │
│ Process · API · A2A · embedded agent adapters                   │
└──────────────┬───────────────────────────────────────────────────┘
               │
┌──────────────▼───────────────────────────────────────────────────┐
│ Tool gateway                                                     │
│ MCP · native tools · secrets · sandbox · receipts · approvals   │
└──────────────────────────────────────────────────────────────────┘

Infrastructure: PostgreSQL · object storage · worker sandboxes
Optional: Redis/NATS, Temporal, OpenTelemetry collector, model gateway
```

### 7.1 Initial implementation choices

- **Control plane:** TypeScript on Node.js
- **API:** Fastify or equivalent standards-based HTTP framework
- **Web:** React with a typed API client
- **Database:** PostgreSQL 16 or newer
- **Query/migrations:** SQL-first migrations with a thin typed query layer
- **Object storage:** S3-compatible API; MinIO for local deployment
- **Queue:** transactional PostgreSQL outbox and `FOR UPDATE SKIP LOCKED` initially
- **Transport:** REST for commands/queries; Server-Sent Events for live events, with WebSocket as a later upgrade (ADR-0004)
- **Policy engine:** Cedar, authored through a constrained YAML surface (section 11.3, ADR-0002)
- **Telemetry:** OpenTelemetry traces, metrics, and structured logs
- **Worker isolation:** containers by default; configurable process isolation for trusted local use
- **Packaging:** Docker Compose first, Kubernetes after the core semantics stabilize

Redis, NATS, Temporal, and a vector database are optional adapters, not version 1 requirements.

### 7.2 Service boundaries

Begin as a modular monolith plus isolated workers. Preserve service boundaries in packages, transactions, and APIs without prematurely requiring distributed deployment. Every module must be usable by an existing stack without adopting the whole system — incremental adoptability is a survival property, not a nicety.

Recommended modules:

- `identity`
- `organization`
- `work`
- `ledger`
- `conversation`
- `decision`
- `artifact`
- `verification`
- `policy`
- `budget`
- `scheduler`
- `runtime-gateway`
- `tool-gateway`
- `memory`
- `notification`

## 8. Agent portability

### 8.1 Runtime contract

```typescript
export interface AgentRuntime {
  discover(): Promise<RuntimeCapabilities>;
  start(assignment: Assignment, context: ContextBundle): Promise<RunHandle>;
  stream(handle: RunHandle): AsyncIterable<RuntimeEvent>;
  steer(handle: RunHandle, input: SteeringInput): Promise<void>;
  checkpoint(handle: RunHandle): Promise<CheckpointRef | null>;
  resume(checkpoint: CheckpointRef): Promise<RunHandle>;
  cancel(handle: RunHandle, reason: string): Promise<void>;
  health(): Promise<RuntimeHealth>;
}
```

Adapters may report that steering, checkpointing, or resumption is unsupported. The scheduler then uses receipts and a new process invocation rather than pretending native resumption exists.

### 8.2 Adapter categories

#### Process adapters

Supervise authenticated local CLIs and harnesses such as Claude Code, Codex, Goose, OpenClaw, and OpenCode. The harness owns its native provider authentication and model selection. CharterOS owns assignment, workspace isolation, environment filtering, lifecycle, policy, event capture, and receipts.

#### Model adapters

Power CharterOS-native agents using local or hosted inference through Ollama, llama.cpp, vLLM, LM Studio, provider APIs, OpenRouter, or an optional LiteLLM gateway.

#### Remote-agent adapters

Discover and delegate to independent agents using A2A (v1.0, with Signed Agent Cards; the protocol absorbed ACP under the Linux Foundation in 2025). Remote agents are treated as contractors with explicit capability, data, cost, and trust boundaries.

#### Workspace bridges (post-v1)

Surface CharterOS work objects inside external human-agent workspaces — Buzz first, given its append-only signed event model — so that conversation happens where teams already are while the work ledger, authority, and budgets remain in CharterOS. A bridge is a projection, never a second source of truth.

### 8.3 Agent manifest

```yaml
apiVersion: charteros.dev/v1alpha1
kind: Agent
metadata:
  name: senior-engineer
spec:
  runtime:
    adapter: claude-code
    command: ["claude", "-p"]
  capabilities:
    - software.implementation
    - software.testing
    - software.review
  workspace:
    strategy: git-worktree
    isolation: container
  permissions:
    tools:
      - filesystem.project.read
      - filesystem.project.write
      - process.test.execute
    networkAllowlist:
      - github.com
      - registry.npmjs.org
  limits:
    taskCostUsd: 8
    taskDuration: 45m
    concurrentRuns: 1
  recovery:
    heartbeatInterval: 15s
    leaseDuration: 60s
    receiptInterval: 5m
```

### 8.4 What portability does and does not mean

Swapping the model behind an agent while keeping its conversational context is a solved problem in 2026 — harnesses do it natively. CharterOS portability is the organizational remainder: the seat's authority, budget, work history, verification record, and in-flight tasks survive the swap, and the swap is a governed event. An occupant change never silently inherits another occupant's credentials, leases, or private runtime state.

### 8.5 Seat handover protocol

Every occupant change — model upgrade, runtime change, principal replacement, human succession — follows one protocol. It is the operational heart of "roles are not agents," and it must cover the hardest case: handover while a task is running under an active lease.

1. **Propose.** A handover is requested (`seat.handover_proposed`), naming the seat, the outgoing and incoming principals, and the reason. Policy determines required approvals; a handover is never below `modify_internal` risk.
2. **Approve.** Required approvals are collected. Self-approval by either principal involved is structurally rejected.
3. **Freeze intake.** The seat stops claiming new tasks. Queued assignments remain with the seat, not the occupant.
4. **Settle in-flight work.** For each active run owned by the outgoing occupant, the scheduler either: (a) requests a final checkpoint and receipt, then cancels the run cleanly; or (b) on an unresponsive occupant, lets the lease expire. Either way the task enters the standard recovery classification (section 9.4). In-flight work is **interrupted and re-dispatched, never silently transferred** — a fresh run under the new occupant begins from receipts and checkpoints, with a fresh lease and fencing token.
5. **Revoke.** All capability grants, short-lived credentials, and tool-gateway sessions issued to the outgoing occupant for this seat are revoked immediately (`capability.revoked`). Revocation is not optional and not deferrable.
6. **Reassign.** The seat assignment record for the outgoing principal is closed; a new assignment opens for the incoming principal (`seat.assigned`). Both are permanent ledger records.
7. **Re-derive authority.** The incoming occupant receives fresh task-scoped grants derived from the **seat's** authority and the active policy set — never copied from the outgoing occupant's accumulated grants. Grant drift dies at every handover.
8. **Resume.** Interrupted tasks re-enter scheduling. The new occupant's first context bundle includes the task's receipts, the handover record, and the reason for the change.
9. **Account.** Budget entries before the handover remain attributed to the outgoing principal; the seat's and task's budgets continue uninterrupted. Cost attribution never blurs across occupants.

Contingencies:

- **Outgoing occupant is dead or hostile:** steps 4–5 degrade gracefully — lease expiry plus immediate revocation make cooperation unnecessary. Fencing tokens guarantee a revoked occupant's late writes are rejected.
- **Incoming occupant fails to start:** the seat remains in `open` intake-freeze; tasks stay `ready` or `interrupted`; escalation fires per the work contracts. Nothing about a failed handover loses work.
- **Handover during reconciliation:** tasks in `reconciliation_required` are settled or explicitly re-owned by a human decision before the new occupant may act on them; an uncertain external side effect is never handed to a principal that lacks the context to reconcile it.
- **Concurrent handover requests:** serialized per seat by unique constraint; the second request fails cleanly.
- **Human seats:** the identical protocol applies. Succession of a human approver is a first-class, auditable event — this is how organizations outlive people, which is the point.

## 9. Scheduling, durability, and recovery

### 9.1 Scheduler responsibilities

The scheduler performs only deterministic control-plane work:

1. Select tasks in `ready` state whose dependencies are satisfied.
2. Confirm the company and, when the task belongs to one, its project are active.
3. Resolve an eligible seat and principal.
4. Evaluate policies and required approvals.
5. Reserve budget.
6. Create a run and acquire a renewable lease.
7. Dispatch an assignment to an appropriate runtime worker.
8. Monitor heartbeats, receipts, deadlines, and cancellation signals.
9. Reconcile final outputs and release unused budget.
10. Route submission to verification or interruption to recovery.

### 9.2 Delivery semantics

Infrastructure provides at-least-once delivery. Correctness comes from:

- idempotency keys on commands and external actions;
- unique constraints for nonrepeatable transitions;
- transactional state changes plus outbox writes;
- renewable leases with fencing tokens;
- immutable receipts;
- reconciliation before retrying uncertain external actions.

The system must never claim universal exactly-once execution.

### 9.3 Run receipts

A receipt is a structured progress record containing:

- completed actions;
- current hypothesis or plan;
- changed internal and external resources;
- produced artifacts;
- verification performed;
- remaining work;
- blockers;
- cost since the prior receipt;
- safe restart instructions;
- whether any action has an uncertain result.

Receipts are self-reported and therefore untrusted until validated: before any recovery decision relies on a receipt, it is cross-checked against the tool-call ledger (section 10.2).

### 9.4 Recovery classifications

- **resumable:** native checkpoint exists and the runtime can resume it;
- **restartable:** no uncertain side effect exists and work can restart from receipts;
- **reconcilable:** an external action may have succeeded and must be inspected;
- **blocked:** required capability, input, approval, or human judgment is missing;
- **terminal:** the task succeeded, failed permanently, or was cancelled.

### 9.5 Acknowledgment and durability contract

“No lost acknowledged work” is a protocol guarantee, not a synonym for “we use a
database.” Each class of work has one observable acknowledgment boundary:

| Class | CharterOS may acknowledge only after | Recovery consequence |
|---|---|---|
| Domain command | Domain mutation, event, and outbox row commit in one PostgreSQL transaction | A returned success is replayable from the ledger even if every process dies before the client receives it |
| Run receipt | Immutable receipt, progress event, and outbox row commit together | Acknowledged progress is available to recovery and context assembly |
| Artifact version | Content is durably stored, its hash and size are verified, then metadata and event commit | A metadata record never points at a partial object; unreferenced uploads are safe garbage-collection candidates |
| External action | The external result, budget effect, result event, and outbox row commit | Until this boundary, a dispatched action may be `uncertain`; it is reconciled, not assumed failed |

API success is emitted after commit. Tokens, logs, subprocess output, WebSocket or
SSE frames, and UI optimism before that commit are **provisional** and carry no
durability claim. If a client loses a response, it repeats the command with the
same idempotency key and receives the committed result.

The guarantee assumes the configured PostgreSQL and object store honor acknowledged
durable writes. A conforming deployment documents its durability settings and runs
the crash suite against the actual storage configuration. CharterOS cannot make a
stronger promise than a dependency that falsely acknowledges persistence.

Object publication uses upload-then-reference ordering:

1. upload to a content-addressed staging key;
2. verify byte length and SHA-256 from stored content;
3. promote or retain the immutable content-addressed object;
4. commit `artifact_versions`, its ledger event, and outbox row;
5. acknowledge the artifact version.

A crash before step 4 can leave an unreferenced object but never a broken artifact
record. Garbage collection removes only objects absent from committed metadata for
longer than a safety interval.

Decision record: ADR-0007.

### 9.6 External-action journal

The tool gateway is the only component permitted to cross a governed external
side-effect boundary. Every such call follows a write-ahead protocol:

1. **Request.** Persist the tool-call intent, redacted arguments, arguments hash,
   risk, capability decision, approval reference, stable idempotency key when
   available, execution deadline, and reconciliation strategy.
2. **Prepare.** Resolve a short-lived credential and validate that the adapter can
   execute the declared reconciliation strategy. Persist `tool.prepared`.
3. **Dispatch.** Commit `status = 'running'` and `tool.started` immediately before
   invoking the provider. No external bytes are sent before this commit.
4. **Record.** Persist the provider result, external operation reference, budget
   effect, `tool.succeeded` or `tool.failed` event, and outbox message before
   acknowledging completion.
5. **Recover.** A `running` call whose deadline passes without a terminal event is
   marked `uncertain`. The scheduler blocks dependent work until reconciliation
   produces a terminal event or a human accepts the uncertainty.

The arguments hash is calculated over RFC 8785 canonical serialization of the
unredacted arguments inside the tool gateway. It proves whether a proposed replay
is byte-for-byte equivalent without retaining secrets in the ledger. The
CharterOS `idempotency_key` deduplicates the command; a separate
`provider_idempotency_key` is sent to a provider that contractually supports it.

Every external call declares exactly one recovery strategy before dispatch:

| Strategy | Permitted recovery |
|---|---|
| `idempotent_replay` | Repeat only the identical request with the identical provider idempotency key |
| `provider_lookup` | Query the provider using its operation or idempotency reference; replay only if the provider proves no operation exists |
| `observe_state` | Inspect the target system for the intended postcondition and record whether it holds |
| `compensating_action` | Reconcile the first action, then execute a separately governed compensation if policy permits |
| `human` | Block and present the complete journal to a human; never retry automatically |
| `not_required` | Allowed only for calls that cannot create an externally visible side effect |

An adapter that cannot state how ambiguity is resolved cannot execute an external
action. “Try again and hope” is not a recovery strategy.

Process harnesses do not receive an exemption. A conforming runtime either routes
governed external actions through the tool gateway or runs in a sandbox whose
network and operating-system policy prevents it from reaching those actions by
another path. A native harness tool that bypasses the journal is not a CharterOS
tool call and cannot be granted consequential authority.

## 10. Verification and acceptance

Verification is where autonomous organizations succeed or become theater. An agent that grades its own homework is the same failure mode as an agent that does the homework wrong — only harder to detect. This section is deliberately as concrete as the recovery model.

### 10.1 The verification ladder

Acceptance criteria in a work contract are bound to verification methods, cheapest first:

1. **Mechanical** — executed by the platform, not by any agent: test suites pass, artifacts exist and match declared hashes, schemas validate, builds compile, linters pass, coverage thresholds hold. Mechanical checks are the only verification the producer may trigger, because their outcome does not depend on anyone's judgment.
2. **Independent agent review** — a different principal reviews the work. For high-risk deliverables the reviewer must be backed by a **different model family** than the producer: two instances of the same model share blind spots, and decorrelating reviewer failure from producer failure is the entire value of review.
3. **Human review** — required by policy tier, not by default for everything (section 11.6). Human attention is the scarcest budget in the system and is spent where risk concentrates.
4. **Sampled audit** — a configurable fraction of auto-accepted and agent-accepted work is re-verified later, by humans or stronger models. Sampling converts "we trust the process" into a measured error rate.

Every rung produces an append-only `task_verifications` record naming the verifier, method, criterion, result, and evidence. Acceptance requires the records demanded by the work contract; the platform enforces producer ≠ verifier for every method except mechanical.

### 10.2 Receipt validation

Receipts drive recovery, and receipts can lie — through model error, prompt injection, or a compromised harness. Before a receipt is trusted:

- every completed action claiming an external effect must correspond to a recorded tool call;
- `uncertain_side_effect: false` is contradicted — and overridden — by any tool call in `uncertain` status;
- claimed artifacts must exist with matching hashes;
- cost deltas must reconcile against budget entries within tolerance.

A receipt that fails validation quarantines the run into `reconciliation_required` and opens an incident. The tool-call ledger is ground truth precisely because the tool gateway, not the agent, writes it.

### 10.3 Separation of duty

- No principal verifies, approves, or accepts its own work, structurally.
- Approval responses from principals that share a runtime or model family with the producer are flagged, and policy may reject them for high-risk actions.
- Verifier assignments rotate; a stable producer-verifier pair across many tasks is itself a signal surfaced for audit.

### 10.4 Acceptance and rework

Acceptance records who accepted, on the basis of which verification records, under which policy version. Rejection returns the task to `ready` with structured reasons attached to the specific acceptance criteria that failed — rework is a measured loop, not a comment thread. Rejection rate and rework depth are first-class metrics (section 23).

## 11. Policy and security model

### 11.1 Action risk levels

- `observe`
- `draft`
- `modify_internal`
- `propose_external`
- `execute_external`
- `irreversible`

### 11.2 Policy example

```yaml
apiVersion: charteros.dev/v1alpha1
kind: Policy
metadata:
  name: external-actions
spec:
  rules:
    - match:
        action: email.send
        resourceScope: external
      require:
        approvals: 1
        approverPrincipalKind: human
    - match:
        action: git.merge
        resource: repository:*/branch/main
      require:
        evidence:
          - tests.passed
          - review.independent
    - match:
        risk: irreversible
      effect: deny
      unless:
        approvals: 2
        includesHuman: true
```

### 11.3 Policy engine: Cedar

The policy language decision is resolved rather than deferred, because its shape leaks into capability grants, permission envelopes, seat authority, and the approval engine — a dozen tables' worth of tentacles that would calcify around whatever engine shipped first.

CharterOS uses **Cedar** as the evaluation engine, authored through the constrained YAML surface above, compiled to Cedar policies at activation time.

- Cedar's principal–action–resource–condition model maps one-to-one onto CharterOS capability grants and resource patterns.
- Cedar is deterministic, total (every evaluation terminates), analyzable (policies can be statically compared and tested), and side-effect free — properties a general-purpose language like Rego does not guarantee by construction.
- A bespoke DSL is rejected: policy languages accrete escape hatches until they become bad programming languages, and Cedar already did the formal-verification work.
- The YAML surface exists so that policies remain reviewable by humans and writable by agents under approval; raw Cedar is an escape hatch for operators, not the authoring default.

Decision record: ADR-0002.

### 11.4 Identity: composed from existing standards

- **Internal principals** hold keypairs; consequential acts are attributable to a principal and, transitively, to a seat assignment.
- **Workers** receive SPIFFE-style workload identities; a worker's identity is bound to its run and lease, so a revoked or expired worker fails closed.
- **Tool credentials** are short-lived, obtained by OAuth 2.1 token exchange (RFC 8693) against the secret provider, scoped to the capability grant that justified them.
- **Remote agents** present A2A Signed Agent Cards; local principal records are created for them with explicit trust levels, and their authority is never wider than the delegating task's permission envelope.
- **MCP** authorization follows the 2026-07-28 specification, including task-scoped sessions.

CharterOS invents no cryptography and no identity format. The novelty is the join: identity → seat → authority → history in one queryable substrate.

### 11.5 Security requirements

- Deny by default.
- Isolate workers by company and run.
- Never put stored provider credentials in a model context.
- Store secret references, never secret values, in PostgreSQL.
- Issue short-lived scoped credentials to workers.
- Require process harnesses to route consequential external actions through the
  tool gateway or prevent those actions at the sandbox boundary.
- Treat retrieved content, messages, tool output, and remote-agent output as untrusted data.
- Require independent review for configured high-risk actions.
- Record all policy decisions and tool calls.
- Redact sensitive fields before writing logs or events.
- Permit immediate credential and capability revocation.
- Sign release artifacts and publish a software bill of materials.
- Support export and deletion policies for company data.

### 11.6 Keeping approvals meaningful

Approval fatigue is a documented failure mode of every human-in-the-loop system: after enough low-signal requests, humans approve everything, and governance becomes the theater this specification refuses to ship. Countermeasures are part of the product, not the deployment guide:

- **Risk-tiered gates.** Only `propose_external` and above route to humans by default. `observe` through `modify_internal` are governed by capability grants and sampled audit, not per-action approval.
- **Provenance-first inbox.** An approval request renders its complete justification chain — task, work contract, evidence, verification records, cost so far, and the exact action with redacted arguments — in one screen. An approver who must go digging will stop digging within a week.
- **Approval budgets.** A company charter states how many human approvals per day the organization is designed to require. The scheduler treats the budget as backpressure: if the queue exceeds it, work queues rather than gates weakening.
- **Rubber-stamp detection.** Approval latency below plausible reading time, sustained 100% approval rates, and approve-without-expanding-evidence are measured and surfaced to the organization itself.
- **Sampled re-review.** A fraction of approved requests are independently re-reviewed; divergence between original approval and audit is an incident, not a curiosity.
- **Policy simulation.** Before activating a policy change, the engine replays recent history to show what would have been blocked or newly allowed — approvers govern the rule, not just the instance.

### 11.7 Threats to address

- Prompt injection through messages, documents, websites, and tool output
- Confused-deputy attacks across tools
- Credential exfiltration
- Malicious or compromised remote agents
- Excessive agency caused by broad tool grants
- Sybil agents and identity impersonation
- Social engineering of authority — the fake CEO email, the forged board decision (the documented Project Vend failure class; countered by ledgered authority: an instruction not in the ledger has no force)
- Event replay and forged callbacks
- Cost-amplification loops
- Approval fatigue as an attack surface — flooding humans with low-risk requests to smuggle a high-risk one
- Infinite delegation and circular task dependencies
- Cross-tenant data leakage
- Artifact tampering
- Approval spoofing
- Concurrent agents modifying the same exclusive resource

## 12. Memory and context assembly

CharterOS distinguishes:

- **run memory:** ephemeral reasoning context and runtime-native session state;
- **task memory:** messages, receipts, evidence, decisions, and artifacts for a task;
- **organizational memory:** charter, policies, people, products, customers, terminology, and prior decisions;
- **derived retrieval indexes:** full-text and semantic indexes rebuilt from authoritative records.

Each run stores a context manifest listing the exact records and artifact versions supplied to the runtime. Context assembly is reproducible and inspectable. Embeddings improve retrieval but never replace the ledger or relational records.

## 13. API conventions

### 13.1 Commands and queries

Mutating endpoints accept `Idempotency-Key`. Successful mutations return the primary resource and resulting event ID. Optimistic concurrency uses an expected resource version.

Example endpoints:

```text
POST   /v1/companies
GET    /v1/companies/{companyId}
POST   /v1/companies/{companyId}/tasks
POST   /v1/companies/{companyId}/objectives
POST   /v1/projects/{projectId}/tasks
POST   /v1/tasks/{taskId}/claim
POST   /v1/tasks/{taskId}/submit
POST   /v1/tasks/{taskId}/verifications
POST   /v1/tasks/{taskId}/accept
POST   /v1/tasks/{taskId}/messages
POST   /v1/seats/{seatId}/handover
POST   /v1/decisions
POST   /v1/approval-requests/{requestId}/respond
GET    /v1/principals/{principalId}/notifications
GET    /v1/companies/{companyId}/events
GET    /v1/companies/{companyId}/stream
POST   /v1/runs/{runId}/receipts
POST   /v1/runs/{runId}/heartbeat
POST   /v1/runs/{runId}/tool-calls
```

### 13.2 Event envelope

```json
{
  "id": "0190f2b0-7f72-7b93-b4e8-93b0bd42e29f",
  "sequence": 1842,
  "companyId": "0190f1c4-1a45-7d40-b2ec-28463d6e01d7",
  "type": "task.submitted",
  "schemaVersion": 1,
  "occurredAt": "2026-07-30T14:08:12.234Z",
  "actorPrincipalId": "0190f21a-00c7-7dc0-b6a4-51fa8dcbdd41",
  "subject": { "type": "task", "id": "0190f297-bdd0-7310-9784-837ff03754f6" },
  "correlationId": "0190f296-a858-7334-a18f-bdfba928d277",
  "causationId": "0190f2a1-dd95-7c8c-b813-9b46d28b00e7",
  "payload": {
    "summary": "Implemented authentication and attached passing test report",
    "artifactIds": ["0190f2ae-d9b0-752a-b98c-97a29a727fa0"]
  }
}
```

## 14. PostgreSQL schema

The following is the baseline logical schema. It is intended to be extracted into ordered SQL migrations. Application migrations should use a dedicated database role; runtime services should use restricted roles. UUIDv7 should be generated by the application until the minimum PostgreSQL version provides an agreed native implementation.

```sql
BEGIN;

CREATE EXTENSION IF NOT EXISTS pgcrypto;
CREATE EXTENSION IF NOT EXISTS citext;

CREATE SCHEMA IF NOT EXISTS charteros;
SET search_path TO charteros, public;

-- Shared trigger for mutable projection tables. Ledger and accounting tables
-- deliberately do not use this trigger because they are append-only.
CREATE OR REPLACE FUNCTION set_updated_at()
RETURNS trigger
LANGUAGE plpgsql
AS $$
BEGIN
  NEW.updated_at = clock_timestamp();
  RETURN NEW;
END;
$$;

CREATE TABLE companies (
  id                  uuid PRIMARY KEY,
  slug                citext NOT NULL UNIQUE,
  name                text NOT NULL,
  description         text,
  status              text NOT NULL DEFAULT 'active'
                      CHECK (status IN ('active', 'paused', 'archived')),
  default_timezone    text NOT NULL DEFAULT 'UTC',
  metadata            jsonb NOT NULL DEFAULT '{}'::jsonb,
  created_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  updated_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  archived_at         timestamptz,
  CHECK (length(btrim(name)) > 0),
  CHECK (jsonb_typeof(metadata) = 'object')
);

CREATE TRIGGER companies_set_updated_at
BEFORE UPDATE ON companies
FOR EACH ROW EXECUTE FUNCTION set_updated_at();

CREATE TABLE principals (
  id                  uuid PRIMARY KEY,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  kind                text NOT NULL CHECK (kind IN ('human', 'agent', 'service')),
  handle              citext NOT NULL,
  display_name        text NOT NULL,
  status              text NOT NULL DEFAULT 'active'
                      CHECK (status IN ('invited', 'active', 'suspended', 'retired')),
  email               citext,
  public_key          text,
  attributes          jsonb NOT NULL DEFAULT '{}'::jsonb,
  created_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  updated_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  retired_at          timestamptz,
  UNIQUE (company_id, handle),
  UNIQUE (company_id, email),
  UNIQUE (company_id, public_key),
  CHECK (length(btrim(display_name)) > 0),
  CHECK (jsonb_typeof(attributes) = 'object')
);

CREATE INDEX principals_company_kind_status_idx
  ON principals (company_id, kind, status);

CREATE TRIGGER principals_set_updated_at
BEFORE UPDATE ON principals
FOR EACH ROW EXECUTE FUNCTION set_updated_at();

CREATE TABLE company_charters (
  id                  uuid PRIMARY KEY,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  version             integer NOT NULL CHECK (version > 0),
  status              text NOT NULL CHECK (status IN ('draft', 'active', 'superseded')),
  mission             text NOT NULL,
  document            jsonb NOT NULL,
  proposed_by         uuid REFERENCES principals(id) ON DELETE RESTRICT,
  activated_by        uuid REFERENCES principals(id) ON DELETE RESTRICT,
  created_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  activated_at        timestamptz,
  superseded_at       timestamptz,
  UNIQUE (company_id, version),
  CHECK (jsonb_typeof(document) = 'object')
);

CREATE UNIQUE INDEX company_charters_one_active_idx
  ON company_charters (company_id)
  WHERE status = 'active';

CREATE TABLE organization_units (
  id                  uuid PRIMARY KEY,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  parent_unit_id      uuid REFERENCES organization_units(id) ON DELETE RESTRICT,
  slug                citext NOT NULL,
  name                text NOT NULL,
  purpose             text,
  status              text NOT NULL DEFAULT 'active'
                      CHECK (status IN ('active', 'archived')),
  created_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  updated_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  UNIQUE (company_id, slug),
  UNIQUE (id, company_id)
);

CREATE TRIGGER organization_units_set_updated_at
BEFORE UPDATE ON organization_units
FOR EACH ROW EXECUTE FUNCTION set_updated_at();

CREATE TABLE roles (
  id                  uuid PRIMARY KEY,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  slug                citext NOT NULL,
  name                text NOT NULL,
  description         text,
  responsibilities    jsonb NOT NULL DEFAULT '[]'::jsonb,
  default_policy      jsonb NOT NULL DEFAULT '{}'::jsonb,
  created_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  updated_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  UNIQUE (company_id, slug),
  UNIQUE (id, company_id),
  CHECK (jsonb_typeof(responsibilities) = 'array'),
  CHECK (jsonb_typeof(default_policy) = 'object')
);

CREATE TRIGGER roles_set_updated_at
BEFORE UPDATE ON roles
FOR EACH ROW EXECUTE FUNCTION set_updated_at();

CREATE TABLE seats (
  id                  uuid PRIMARY KEY,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  organization_unit_id uuid,
  role_id             uuid NOT NULL,
  reports_to_seat_id  uuid REFERENCES seats(id) ON DELETE SET NULL,
  slug                citext NOT NULL,
  title               text NOT NULL,
  status              text NOT NULL DEFAULT 'open'
                      CHECK (status IN ('open', 'occupied', 'suspended', 'closed')),
  authority           jsonb NOT NULL DEFAULT '{}'::jsonb,
  created_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  updated_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  UNIQUE (company_id, slug),
  UNIQUE (id, company_id),
  FOREIGN KEY (organization_unit_id, company_id)
    REFERENCES organization_units(id, company_id) ON DELETE RESTRICT,
  FOREIGN KEY (role_id, company_id)
    REFERENCES roles(id, company_id) ON DELETE RESTRICT,
  CHECK (jsonb_typeof(authority) = 'object'),
  CHECK (reports_to_seat_id IS NULL OR reports_to_seat_id <> id)
);

CREATE INDEX seats_company_unit_idx
  ON seats (company_id, organization_unit_id, status);

CREATE TRIGGER seats_set_updated_at
BEFORE UPDATE ON seats
FOR EACH ROW EXECUTE FUNCTION set_updated_at();

CREATE TABLE seat_assignments (
  id                  uuid PRIMARY KEY,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  seat_id             uuid NOT NULL,
  principal_id        uuid NOT NULL REFERENCES principals(id) ON DELETE RESTRICT,
  assignment_kind     text NOT NULL DEFAULT 'primary'
                      CHECK (assignment_kind IN ('primary', 'acting', 'delegate')),
  starts_at           timestamptz NOT NULL DEFAULT clock_timestamp(),
  ends_at             timestamptz,
  assigned_by         uuid REFERENCES principals(id) ON DELETE RESTRICT,
  reason              text,
  created_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  FOREIGN KEY (seat_id, company_id)
    REFERENCES seats(id, company_id) ON DELETE CASCADE,
  CHECK (ends_at IS NULL OR ends_at > starts_at)
);

CREATE UNIQUE INDEX seat_assignments_one_current_primary_idx
  ON seat_assignments (seat_id)
  WHERE ends_at IS NULL AND assignment_kind = 'primary';

CREATE INDEX seat_assignments_principal_current_idx
  ON seat_assignments (company_id, principal_id)
  WHERE ends_at IS NULL;

CREATE TABLE runtime_definitions (
  id                  uuid PRIMARY KEY,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  slug                citext NOT NULL,
  adapter_type        text NOT NULL
                      CHECK (adapter_type IN ('process', 'model', 'acp', 'a2a', 'custom')),
  adapter_name        text NOT NULL,
  endpoint            text,
  command             jsonb,
  configuration       jsonb NOT NULL DEFAULT '{}'::jsonb,
  capabilities        jsonb NOT NULL DEFAULT '[]'::jsonb,
  status              text NOT NULL DEFAULT 'active'
                      CHECK (status IN ('active', 'disabled', 'unhealthy')),
  created_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  updated_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  UNIQUE (company_id, slug),
  UNIQUE (id, company_id),
  CHECK (command IS NULL OR jsonb_typeof(command) = 'array'),
  CHECK (jsonb_typeof(configuration) = 'object'),
  CHECK (jsonb_typeof(capabilities) = 'array')
);

CREATE TRIGGER runtime_definitions_set_updated_at
BEFORE UPDATE ON runtime_definitions
FOR EACH ROW EXECUTE FUNCTION set_updated_at();

CREATE TABLE agent_profiles (
  principal_id        uuid PRIMARY KEY REFERENCES principals(id) ON DELETE CASCADE,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  runtime_definition_id uuid,
  model_ref           text,
  persona             jsonb NOT NULL DEFAULT '{}'::jsonb,
  limits              jsonb NOT NULL DEFAULT '{}'::jsonb,
  recovery_policy     jsonb NOT NULL DEFAULT '{}'::jsonb,
  created_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  updated_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  FOREIGN KEY (runtime_definition_id, company_id)
    REFERENCES runtime_definitions(id, company_id) ON DELETE RESTRICT,
  CHECK (jsonb_typeof(persona) = 'object'),
  CHECK (jsonb_typeof(limits) = 'object'),
  CHECK (jsonb_typeof(recovery_policy) = 'object')
);

CREATE TRIGGER agent_profiles_set_updated_at
BEFORE UPDATE ON agent_profiles
FOR EACH ROW EXECUTE FUNCTION set_updated_at();

CREATE TABLE objectives (
  id                  uuid PRIMARY KEY,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  parent_objective_id uuid REFERENCES objectives(id) ON DELETE RESTRICT,
  owner_seat_id       uuid,
  title               text NOT NULL,
  description         text,
  status              text NOT NULL DEFAULT 'draft'
                      CHECK (status IN ('draft', 'proposed', 'active', 'achieved', 'abandoned')),
  success_metrics     jsonb NOT NULL DEFAULT '[]'::jsonb,
  target_at           timestamptz,
  created_by          uuid NOT NULL REFERENCES principals(id) ON DELETE RESTRICT,
  created_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  updated_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  closed_at           timestamptz,
  version             integer NOT NULL DEFAULT 1 CHECK (version > 0),
  UNIQUE (id, company_id),
  FOREIGN KEY (owner_seat_id, company_id)
    REFERENCES seats(id, company_id) ON DELETE RESTRICT,
  CHECK (jsonb_typeof(success_metrics) = 'array')
);

CREATE INDEX objectives_company_status_idx
  ON objectives (company_id, status, target_at);

CREATE TRIGGER objectives_set_updated_at
BEFORE UPDATE ON objectives
FOR EACH ROW EXECUTE FUNCTION set_updated_at();

CREATE TABLE projects (
  id                  uuid PRIMARY KEY,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  objective_id        uuid,
  owner_seat_id       uuid,
  slug                citext NOT NULL,
  name                text NOT NULL,
  description         text,
  status              text NOT NULL DEFAULT 'draft'
                      CHECK (status IN ('draft', 'active', 'paused', 'completed', 'cancelled', 'archived')),
  repository_url      text,
  settings            jsonb NOT NULL DEFAULT '{}'::jsonb,
  starts_at           timestamptz,
  due_at              timestamptz,
  created_by          uuid NOT NULL REFERENCES principals(id) ON DELETE RESTRICT,
  created_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  updated_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  completed_at        timestamptz,
  version             integer NOT NULL DEFAULT 1 CHECK (version > 0),
  UNIQUE (company_id, slug),
  UNIQUE (id, company_id),
  FOREIGN KEY (objective_id, company_id)
    REFERENCES objectives(id, company_id) ON DELETE RESTRICT,
  FOREIGN KEY (owner_seat_id, company_id)
    REFERENCES seats(id, company_id) ON DELETE RESTRICT,
  CHECK (jsonb_typeof(settings) = 'object'),
  CHECK (due_at IS NULL OR starts_at IS NULL OR due_at >= starts_at)
);

CREATE INDEX projects_company_status_idx
  ON projects (company_id, status, due_at);

CREATE TRIGGER projects_set_updated_at
BEFORE UPDATE ON projects
FOR EACH ROW EXECUTE FUNCTION set_updated_at();

CREATE TABLE tasks (
  id                  uuid PRIMARY KEY,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  project_id          uuid,
  parent_task_id      uuid REFERENCES tasks(id) ON DELETE RESTRICT,
  requester_principal_id uuid NOT NULL REFERENCES principals(id) ON DELETE RESTRICT,
  owner_seat_id       uuid,
  owner_principal_id  uuid REFERENCES principals(id) ON DELETE RESTRICT,
  title               text NOT NULL,
  description         text,
  status              text NOT NULL DEFAULT 'draft'
                      CHECK (status IN (
                        'draft', 'proposed', 'ready', 'claimed', 'running',
                        'submitted', 'verifying', 'accepted', 'closed',
                        'blocked', 'interrupted', 'reconciliation_required',
                        'failed', 'cancelled'
                      )),
  priority            smallint NOT NULL DEFAULT 50 CHECK (priority BETWEEN 0 AND 100),
  input_spec          jsonb NOT NULL DEFAULT '{}'::jsonb,
  deliverables        jsonb NOT NULL DEFAULT '[]'::jsonb,
  acceptance_criteria jsonb NOT NULL DEFAULT '[]'::jsonb,
  permission_envelope jsonb NOT NULL DEFAULT '{}'::jsonb,
  retry_policy        jsonb NOT NULL DEFAULT '{}'::jsonb,
  max_cost_microusd   bigint CHECK (max_cost_microusd IS NULL OR max_cost_microusd >= 0),
  max_tokens          bigint CHECK (max_tokens IS NULL OR max_tokens >= 0),
  max_duration_seconds integer CHECK (max_duration_seconds IS NULL OR max_duration_seconds > 0),
  not_before          timestamptz,
  due_at              timestamptz,
  created_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  updated_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  submitted_at        timestamptz,
  accepted_at         timestamptz,
  closed_at           timestamptz,
  version             integer NOT NULL DEFAULT 1 CHECK (version > 0),
  UNIQUE (id, company_id),
  FOREIGN KEY (project_id, company_id)
    REFERENCES projects(id, company_id) ON DELETE CASCADE,
  FOREIGN KEY (owner_seat_id, company_id)
    REFERENCES seats(id, company_id) ON DELETE RESTRICT,
  CHECK (jsonb_typeof(input_spec) = 'object'),
  CHECK (jsonb_typeof(deliverables) = 'array'),
  CHECK (jsonb_typeof(acceptance_criteria) = 'array'),
  CHECK (jsonb_typeof(permission_envelope) = 'object'),
  CHECK (jsonb_typeof(retry_policy) = 'object')
);

CREATE INDEX tasks_scheduler_idx
  ON tasks (company_id, status, priority DESC, not_before, created_at)
  WHERE status IN ('ready', 'interrupted');

CREATE INDEX tasks_project_status_idx
  ON tasks (project_id, status, updated_at DESC);

CREATE INDEX tasks_owner_idx
  ON tasks (company_id, owner_principal_id, status);

CREATE TRIGGER tasks_set_updated_at
BEFORE UPDATE ON tasks
FOR EACH ROW EXECUTE FUNCTION set_updated_at();

CREATE TABLE task_dependencies (
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  task_id             uuid NOT NULL,
  depends_on_task_id  uuid NOT NULL,
  dependency_type     text NOT NULL DEFAULT 'blocks'
                      CHECK (dependency_type IN ('blocks', 'requires_output', 'related')),
  created_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  PRIMARY KEY (task_id, depends_on_task_id),
  FOREIGN KEY (task_id, company_id)
    REFERENCES tasks(id, company_id) ON DELETE CASCADE,
  FOREIGN KEY (depends_on_task_id, company_id)
    REFERENCES tasks(id, company_id) ON DELETE CASCADE,
  CHECK (task_id <> depends_on_task_id)
);

CREATE INDEX task_dependencies_reverse_idx
  ON task_dependencies (depends_on_task_id, task_id);

CREATE TABLE conversations (
  id                  uuid PRIMARY KEY,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  subject_type        text NOT NULL CHECK (subject_type IN (
                        'company', 'objective', 'project', 'task', 'decision', 'artifact', 'incident'
                      )),
  subject_id          uuid NOT NULL,
  title               text,
  visibility          text NOT NULL DEFAULT 'company'
                      CHECK (visibility IN ('company', 'restricted', 'private')),
  created_by          uuid NOT NULL REFERENCES principals(id) ON DELETE RESTRICT,
  created_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  archived_at         timestamptz,
  UNIQUE (company_id, subject_type, subject_id),
  UNIQUE (id, company_id)
);

CREATE TABLE conversation_members (
  conversation_id     uuid NOT NULL,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  principal_id        uuid NOT NULL REFERENCES principals(id) ON DELETE CASCADE,
  membership_role     text NOT NULL DEFAULT 'member'
                      CHECK (membership_role IN ('owner', 'member', 'observer')),
  joined_at           timestamptz NOT NULL DEFAULT clock_timestamp(),
  left_at             timestamptz,
  PRIMARY KEY (conversation_id, principal_id),
  FOREIGN KEY (conversation_id, company_id)
    REFERENCES conversations(id, company_id) ON DELETE CASCADE
);

CREATE TABLE messages (
  id                  uuid PRIMARY KEY,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  conversation_id     uuid NOT NULL,
  parent_message_id   uuid,
  author_principal_id uuid NOT NULL REFERENCES principals(id) ON DELETE RESTRICT,
  body_markdown       text NOT NULL,
  structured_content  jsonb NOT NULL DEFAULT '{}'::jsonb,
  message_type        text NOT NULL DEFAULT 'comment'
                      CHECK (message_type IN ('comment', 'status', 'question', 'answer', 'system')),
  created_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  superseded_by       uuid,
  redacted_at         timestamptz,
  UNIQUE (id, company_id),
  FOREIGN KEY (conversation_id, company_id)
    REFERENCES conversations(id, company_id) ON DELETE CASCADE,
  FOREIGN KEY (parent_message_id, company_id)
    REFERENCES messages(id, company_id) ON DELETE RESTRICT,
  FOREIGN KEY (superseded_by, company_id)
    REFERENCES messages(id, company_id) ON DELETE RESTRICT,
  CHECK (length(body_markdown) > 0 OR structured_content <> '{}'::jsonb),
  CHECK (jsonb_typeof(structured_content) = 'object')
);

CREATE INDEX messages_conversation_created_idx
  ON messages (conversation_id, created_at, id);

CREATE INDEX messages_search_idx
  ON messages USING gin (to_tsvector('english', body_markdown));

CREATE TABLE artifacts (
  id                  uuid PRIMARY KEY,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  project_id          uuid,
  task_id             uuid,
  kind                text NOT NULL,
  name                text NOT NULL,
  description         text,
  current_version     integer NOT NULL DEFAULT 0 CHECK (current_version >= 0),
  status              text NOT NULL DEFAULT 'active'
                      CHECK (status IN ('active', 'superseded', 'archived', 'quarantined')),
  created_by          uuid NOT NULL REFERENCES principals(id) ON DELETE RESTRICT,
  created_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  updated_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  UNIQUE (id, company_id),
  FOREIGN KEY (project_id, company_id)
    REFERENCES projects(id, company_id) ON DELETE CASCADE,
  FOREIGN KEY (task_id, company_id)
    REFERENCES tasks(id, company_id) ON DELETE CASCADE
);

CREATE TRIGGER artifacts_set_updated_at
BEFORE UPDATE ON artifacts
FOR EACH ROW EXECUTE FUNCTION set_updated_at();

CREATE TABLE artifact_versions (
  id                  uuid PRIMARY KEY,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  artifact_id         uuid NOT NULL,
  version             integer NOT NULL CHECK (version > 0),
  storage_uri         text NOT NULL,
  media_type          text NOT NULL,
  byte_size           bigint CHECK (byte_size IS NULL OR byte_size >= 0),
  sha256              text NOT NULL CHECK (sha256 ~ '^[0-9a-f]{64}$'),
  metadata            jsonb NOT NULL DEFAULT '{}'::jsonb,
  created_by          uuid NOT NULL REFERENCES principals(id) ON DELETE RESTRICT,
  created_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  UNIQUE (artifact_id, version),
  UNIQUE (company_id, sha256, storage_uri),
  UNIQUE (id, company_id),
  FOREIGN KEY (artifact_id, company_id)
    REFERENCES artifacts(id, company_id) ON DELETE CASCADE,
  CHECK (jsonb_typeof(metadata) = 'object')
);

CREATE TABLE evidence_links (
  id                  uuid PRIMARY KEY,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  artifact_version_id uuid,
  source_uri          text,
  claim               text,
  relation            text NOT NULL DEFAULT 'supports'
                      CHECK (relation IN ('supports', 'opposes', 'verifies', 'produced_by')),
  created_by          uuid NOT NULL REFERENCES principals(id) ON DELETE RESTRICT,
  created_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  UNIQUE (id, company_id),
  FOREIGN KEY (artifact_version_id, company_id)
    REFERENCES artifact_versions(id, company_id) ON DELETE RESTRICT,
  CHECK (artifact_version_id IS NOT NULL OR source_uri IS NOT NULL)
);

CREATE TABLE decisions (
  id                  uuid PRIMARY KEY,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  project_id          uuid,
  task_id             uuid,
  owner_principal_id  uuid NOT NULL REFERENCES principals(id) ON DELETE RESTRICT,
  title               text NOT NULL,
  question            text NOT NULL,
  alternatives        jsonb NOT NULL DEFAULT '[]'::jsonb,
  status              text NOT NULL DEFAULT 'open'
                      CHECK (status IN ('open', 'proposed', 'approved', 'rejected', 'superseded', 'withdrawn')),
  resolution          jsonb,
  rationale_markdown  text,
  supersedes_decision_id uuid REFERENCES decisions(id) ON DELETE RESTRICT,
  due_at              timestamptz,
  resolved_at         timestamptz,
  created_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  updated_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  version             integer NOT NULL DEFAULT 1 CHECK (version > 0),
  UNIQUE (id, company_id),
  FOREIGN KEY (project_id, company_id)
    REFERENCES projects(id, company_id) ON DELETE CASCADE,
  FOREIGN KEY (task_id, company_id)
    REFERENCES tasks(id, company_id) ON DELETE CASCADE,
  CHECK (jsonb_typeof(alternatives) = 'array'),
  CHECK (resolution IS NULL OR jsonb_typeof(resolution) = 'object')
);

CREATE INDEX decisions_company_status_idx
  ON decisions (company_id, status, due_at);

CREATE TRIGGER decisions_set_updated_at
BEFORE UPDATE ON decisions
FOR EACH ROW EXECUTE FUNCTION set_updated_at();

CREATE TABLE decision_evidence (
  decision_id         uuid NOT NULL,
  evidence_link_id    uuid NOT NULL,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  created_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  PRIMARY KEY (decision_id, evidence_link_id),
  FOREIGN KEY (decision_id, company_id)
    REFERENCES decisions(id, company_id) ON DELETE CASCADE,
  FOREIGN KEY (evidence_link_id, company_id)
    REFERENCES evidence_links(id, company_id) ON DELETE RESTRICT
);

CREATE TABLE policies (
  id                  uuid PRIMARY KEY,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  slug                citext NOT NULL,
  name                text NOT NULL,
  version             integer NOT NULL CHECK (version > 0),
  status              text NOT NULL DEFAULT 'draft'
                      CHECK (status IN ('draft', 'active', 'superseded', 'disabled')),
  policy_document     jsonb NOT NULL,
  created_by          uuid NOT NULL REFERENCES principals(id) ON DELETE RESTRICT,
  created_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  activated_at        timestamptz,
  UNIQUE (company_id, slug, version),
  CHECK (jsonb_typeof(policy_document) = 'object')
);

CREATE UNIQUE INDEX policies_one_active_version_idx
  ON policies (company_id, slug)
  WHERE status = 'active';

CREATE TABLE approval_requests (
  id                  uuid PRIMARY KEY,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  subject_type        text NOT NULL,
  subject_id          uuid NOT NULL,
  action_type         text NOT NULL,
  policy_id           uuid REFERENCES policies(id) ON DELETE RESTRICT,
  requested_by        uuid NOT NULL REFERENCES principals(id) ON DELETE RESTRICT,
  status              text NOT NULL DEFAULT 'pending'
                      CHECK (status IN ('pending', 'approved', 'rejected', 'expired', 'cancelled')),
  required_approvals  integer NOT NULL DEFAULT 1 CHECK (required_approvals > 0),
  constraints         jsonb NOT NULL DEFAULT '{}'::jsonb,
  expires_at          timestamptz,
  created_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  resolved_at         timestamptz,
  UNIQUE (id, company_id),
  CHECK (jsonb_typeof(constraints) = 'object')
);

CREATE INDEX approval_requests_pending_idx
  ON approval_requests (company_id, expires_at, created_at)
  WHERE status = 'pending';

CREATE TABLE approval_responses (
  id                  uuid PRIMARY KEY,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  approval_request_id uuid NOT NULL,
  responder_principal_id uuid NOT NULL REFERENCES principals(id) ON DELETE RESTRICT,
  response            text NOT NULL CHECK (response IN ('approve', 'reject', 'abstain')),
  reason              text,
  conditions          jsonb NOT NULL DEFAULT '{}'::jsonb,
  created_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  UNIQUE (approval_request_id, responder_principal_id),
  FOREIGN KEY (approval_request_id, company_id)
    REFERENCES approval_requests(id, company_id) ON DELETE CASCADE,
  CHECK (jsonb_typeof(conditions) = 'object')
);

CREATE TABLE budgets (
  id                  uuid PRIMARY KEY,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  parent_budget_id    uuid REFERENCES budgets(id) ON DELETE RESTRICT,
  scope_type          text NOT NULL CHECK (scope_type IN ('company', 'unit', 'project', 'task', 'principal')),
  scope_id            uuid NOT NULL,
  currency            char(3) NOT NULL DEFAULT 'USD',
  limit_microunits    bigint NOT NULL CHECK (limit_microunits >= 0),
  period              text NOT NULL DEFAULT 'lifetime'
                      CHECK (period IN ('task', 'day', 'week', 'month', 'quarter', 'year', 'lifetime')),
  period_starts_at    timestamptz,
  period_ends_at      timestamptz,
  status              text NOT NULL DEFAULT 'active'
                      CHECK (status IN ('active', 'exhausted', 'closed')),
  created_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  updated_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  UNIQUE (id, company_id),
  CHECK (period_ends_at IS NULL OR period_starts_at IS NULL OR period_ends_at > period_starts_at)
);

CREATE INDEX budgets_scope_idx
  ON budgets (company_id, scope_type, scope_id, status);

CREATE TRIGGER budgets_set_updated_at
BEFORE UPDATE ON budgets
FOR EACH ROW EXECUTE FUNCTION set_updated_at();

CREATE TABLE budget_entries (
  id                  uuid PRIMARY KEY,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  budget_id           uuid NOT NULL,
  entry_type          text NOT NULL
                      CHECK (entry_type IN ('reserve', 'release', 'debit', 'credit', 'adjustment')),
  amount_microunits   bigint NOT NULL CHECK (amount_microunits >= 0),
  run_id              uuid,
  task_id             uuid,
  provider_ref        text,
  model_ref           text,
  input_tokens        bigint CHECK (input_tokens IS NULL OR input_tokens >= 0),
  output_tokens       bigint CHECK (output_tokens IS NULL OR output_tokens >= 0),
  idempotency_key     text NOT NULL,
  metadata            jsonb NOT NULL DEFAULT '{}'::jsonb,
  occurred_at         timestamptz NOT NULL DEFAULT clock_timestamp(),
  UNIQUE (company_id, idempotency_key),
  FOREIGN KEY (budget_id, company_id)
    REFERENCES budgets(id, company_id) ON DELETE RESTRICT,
  CHECK (jsonb_typeof(metadata) = 'object')
);

CREATE INDEX budget_entries_budget_time_idx
  ON budget_entries (budget_id, occurred_at, id);

CREATE TABLE runs (
  id                  uuid PRIMARY KEY,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  task_id             uuid NOT NULL,
  principal_id        uuid NOT NULL REFERENCES principals(id) ON DELETE RESTRICT,
  runtime_definition_id uuid,
  charter_id          uuid REFERENCES company_charters(id) ON DELETE RESTRICT,
  attempt             integer NOT NULL CHECK (attempt > 0),
  status              text NOT NULL DEFAULT 'queued'
                      CHECK (status IN (
                        'queued', 'starting', 'running', 'suspended', 'submitted',
                        'succeeded', 'failed', 'cancelled', 'timed_out',
                        'interrupted', 'reconciliation_required'
                      )),
  context_manifest    jsonb NOT NULL DEFAULT '{}'::jsonb,
  runtime_metadata    jsonb NOT NULL DEFAULT '{}'::jsonb,
  started_at          timestamptz,
  last_heartbeat_at   timestamptz,
  finished_at         timestamptz,
  exit_code           integer,
  failure_code        text,
  failure_detail      text,
  created_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  updated_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  UNIQUE (task_id, attempt),
  UNIQUE (id, company_id),
  FOREIGN KEY (task_id, company_id)
    REFERENCES tasks(id, company_id) ON DELETE CASCADE,
  FOREIGN KEY (runtime_definition_id, company_id)
    REFERENCES runtime_definitions(id, company_id) ON DELETE RESTRICT,
  CHECK (jsonb_typeof(context_manifest) = 'object'),
  CHECK (jsonb_typeof(runtime_metadata) = 'object'),
  CHECK (finished_at IS NULL OR started_at IS NULL OR finished_at >= started_at)
);

ALTER TABLE budget_entries
  ADD CONSTRAINT budget_entries_run_company_fk
  FOREIGN KEY (run_id, company_id) REFERENCES runs(id, company_id) ON DELETE RESTRICT;

ALTER TABLE budget_entries
  ADD CONSTRAINT budget_entries_task_company_fk
  FOREIGN KEY (task_id, company_id) REFERENCES tasks(id, company_id) ON DELETE RESTRICT;

CREATE INDEX runs_active_idx
  ON runs (company_id, status, last_heartbeat_at)
  WHERE status IN ('queued', 'starting', 'running', 'suspended');

CREATE INDEX runs_task_idx
  ON runs (task_id, attempt DESC);

CREATE TRIGGER runs_set_updated_at
BEFORE UPDATE ON runs
FOR EACH ROW EXECUTE FUNCTION set_updated_at();

CREATE TABLE run_leases (
  run_id              uuid PRIMARY KEY,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  worker_id           text NOT NULL,
  fencing_token       bigint NOT NULL CHECK (fencing_token > 0),
  acquired_at         timestamptz NOT NULL DEFAULT clock_timestamp(),
  heartbeat_at        timestamptz NOT NULL DEFAULT clock_timestamp(),
  expires_at          timestamptz NOT NULL,
  FOREIGN KEY (run_id, company_id)
    REFERENCES runs(id, company_id) ON DELETE CASCADE,
  CHECK (expires_at > acquired_at)
);

CREATE INDEX run_leases_expiry_idx ON run_leases (expires_at);

CREATE TABLE run_receipts (
  id                  uuid PRIMARY KEY,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  run_id              uuid NOT NULL,
  sequence            integer NOT NULL CHECK (sequence > 0),
  receipt_type        text NOT NULL DEFAULT 'progress'
                      CHECK (receipt_type IN ('progress', 'checkpoint', 'handoff', 'final', 'reconciliation')),
  summary             text NOT NULL,
  completed_actions   jsonb NOT NULL DEFAULT '[]'::jsonb,
  changed_resources   jsonb NOT NULL DEFAULT '[]'::jsonb,
  artifact_refs       jsonb NOT NULL DEFAULT '[]'::jsonb,
  verification        jsonb NOT NULL DEFAULT '[]'::jsonb,
  remaining_work      jsonb NOT NULL DEFAULT '[]'::jsonb,
  blockers            jsonb NOT NULL DEFAULT '[]'::jsonb,
  restart_instructions text,
  uncertain_side_effect boolean NOT NULL DEFAULT false,
  cost_delta          jsonb NOT NULL DEFAULT '{}'::jsonb,
  created_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  UNIQUE (run_id, sequence),
  UNIQUE (id, company_id),
  FOREIGN KEY (run_id, company_id)
    REFERENCES runs(id, company_id) ON DELETE CASCADE,
  CHECK (jsonb_typeof(completed_actions) = 'array'),
  CHECK (jsonb_typeof(changed_resources) = 'array'),
  CHECK (jsonb_typeof(artifact_refs) = 'array'),
  CHECK (jsonb_typeof(verification) = 'array'),
  CHECK (jsonb_typeof(remaining_work) = 'array'),
  CHECK (jsonb_typeof(blockers) = 'array'),
  CHECK (jsonb_typeof(cost_delta) = 'object')
);

CREATE INDEX run_receipts_run_created_idx
  ON run_receipts (run_id, sequence DESC);

CREATE TABLE run_checkpoints (
  id                  uuid PRIMARY KEY,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  run_id              uuid NOT NULL,
  receipt_id          uuid,
  runtime_checkpoint_ref text NOT NULL,
  resumable           boolean NOT NULL DEFAULT false,
  metadata            jsonb NOT NULL DEFAULT '{}'::jsonb,
  created_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  FOREIGN KEY (run_id, company_id)
    REFERENCES runs(id, company_id) ON DELETE CASCADE,
  FOREIGN KEY (receipt_id, company_id)
    REFERENCES run_receipts(id, company_id) ON DELETE RESTRICT,
  CHECK (jsonb_typeof(metadata) = 'object')
);

CREATE INDEX run_checkpoints_run_created_idx
  ON run_checkpoints (run_id, created_at DESC);

CREATE TABLE capability_grants (
  id                  uuid PRIMARY KEY,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  principal_id        uuid NOT NULL REFERENCES principals(id) ON DELETE CASCADE,
  run_id              uuid,
  capability          text NOT NULL,
  resource_pattern    text NOT NULL,
  effect              text NOT NULL DEFAULT 'allow' CHECK (effect IN ('allow', 'deny')),
  constraints         jsonb NOT NULL DEFAULT '{}'::jsonb,
  granted_by          uuid NOT NULL REFERENCES principals(id) ON DELETE RESTRICT,
  issued_at           timestamptz NOT NULL DEFAULT clock_timestamp(),
  expires_at          timestamptz,
  revoked_at          timestamptz,
  FOREIGN KEY (run_id, company_id)
    REFERENCES runs(id, company_id) ON DELETE CASCADE,
  CHECK (expires_at IS NULL OR expires_at > issued_at),
  CHECK (jsonb_typeof(constraints) = 'object')
);

CREATE INDEX capability_grants_lookup_idx
  ON capability_grants (company_id, principal_id, capability)
  WHERE revoked_at IS NULL;

CREATE TABLE secret_references (
  id                  uuid PRIMARY KEY,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  name                citext NOT NULL,
  provider            text NOT NULL,
  provider_reference  text NOT NULL,
  allowed_capabilities jsonb NOT NULL DEFAULT '[]'::jsonb,
  created_by          uuid NOT NULL REFERENCES principals(id) ON DELETE RESTRICT,
  created_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  rotated_at          timestamptz,
  revoked_at          timestamptz,
  UNIQUE (company_id, name),
  CHECK (jsonb_typeof(allowed_capabilities) = 'array')
);

CREATE TABLE tool_calls (
  id                  uuid PRIMARY KEY,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  run_id              uuid NOT NULL,
  principal_id        uuid NOT NULL REFERENCES principals(id) ON DELETE RESTRICT,
  tool_server         text NOT NULL,
  tool_name           text NOT NULL,
  risk_level          text NOT NULL CHECK (risk_level IN (
                        'observe', 'draft', 'modify_internal',
                        'propose_external', 'execute_external', 'irreversible'
                      )),
  idempotency_key     text,
  provider_idempotency_key text,
  arguments_sha256    text NOT NULL CHECK (arguments_sha256 ~ '^[0-9a-f]{64}$'),
  arguments_redacted  jsonb NOT NULL DEFAULT '{}'::jsonb,
  result_redacted     jsonb,
  reconciliation_strategy text NOT NULL DEFAULT 'not_required'
                      CHECK (reconciliation_strategy IN (
                        'not_required', 'idempotent_replay', 'provider_lookup',
                        'observe_state', 'compensating_action', 'human'
                      )),
  reconciliation_spec jsonb NOT NULL DEFAULT '{}'::jsonb,
  status              text NOT NULL DEFAULT 'requested'
                      CHECK (status IN (
                        'requested', 'awaiting_approval', 'prepared', 'running',
                        'succeeded', 'failed', 'denied', 'uncertain'
                      )),
  approval_request_id uuid,
  external_operation_ref text,
  requested_at        timestamptz NOT NULL DEFAULT clock_timestamp(),
  prepared_at         timestamptz,
  started_at          timestamptz,
  execution_deadline_at timestamptz,
  finished_at         timestamptz,
  reconciled_at       timestamptz,
  UNIQUE (id, company_id),
  FOREIGN KEY (run_id, company_id)
    REFERENCES runs(id, company_id) ON DELETE CASCADE,
  FOREIGN KEY (approval_request_id, company_id)
    REFERENCES approval_requests(id, company_id) ON DELETE RESTRICT,
  CHECK (jsonb_typeof(arguments_redacted) = 'object'),
  CHECK (jsonb_typeof(reconciliation_spec) = 'object'),
  CHECK (result_redacted IS NULL OR jsonb_typeof(result_redacted) IN ('object', 'array', 'string', 'number', 'boolean', 'null')),
  CHECK (
    risk_level NOT IN ('propose_external', 'execute_external', 'irreversible')
    OR reconciliation_strategy <> 'not_required'
  ),
  CHECK (
    reconciliation_strategy <> 'idempotent_replay'
    OR provider_idempotency_key IS NOT NULL
  ),
  CHECK (
    status NOT IN ('prepared', 'running') OR prepared_at IS NOT NULL
  ),
  CHECK (
    status <> 'running'
    OR (started_at IS NOT NULL AND execution_deadline_at > started_at)
  ),
  CHECK (
    status NOT IN ('succeeded', 'uncertain')
    OR (started_at IS NOT NULL AND execution_deadline_at > started_at)
  ),
  CHECK (
    status NOT IN ('succeeded', 'failed', 'denied') OR finished_at IS NOT NULL
  ),
  CHECK (reconciled_at IS NULL OR status IN ('succeeded', 'failed', 'uncertain'))
);

CREATE INDEX tool_calls_run_requested_idx
  ON tool_calls (run_id, requested_at, id);

CREATE INDEX tool_calls_uncertain_idx
  ON tool_calls (company_id, requested_at)
  WHERE status = 'uncertain';

CREATE INDEX tool_calls_recovery_idx
  ON tool_calls (company_id, execution_deadline_at)
  WHERE status = 'running';

CREATE UNIQUE INDEX tool_calls_idempotency_idx
  ON tool_calls (company_id, tool_server, tool_name, idempotency_key)
  WHERE idempotency_key IS NOT NULL;

-- Verification records are first-class evidence: acceptance requires them, and
-- they are append-only like receipts. The verifier must not be the producer
-- except for platform-executed mechanical checks (enforced in the domain layer).
CREATE TABLE task_verifications (
  id                  uuid PRIMARY KEY,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  task_id             uuid NOT NULL,
  run_id              uuid,
  verifier_principal_id uuid NOT NULL REFERENCES principals(id) ON DELETE RESTRICT,
  method              text NOT NULL CHECK (method IN (
                        'mechanical', 'agent_review', 'human_review', 'sampled_audit'
                      )),
  criterion_ref       text NOT NULL,
  result              text NOT NULL CHECK (result IN ('passed', 'failed', 'inconclusive')),
  detail_markdown     text,
  evidence            jsonb NOT NULL DEFAULT '[]'::jsonb,
  created_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  UNIQUE (id, company_id),
  FOREIGN KEY (task_id, company_id)
    REFERENCES tasks(id, company_id) ON DELETE CASCADE,
  FOREIGN KEY (run_id, company_id)
    REFERENCES runs(id, company_id) ON DELETE SET NULL (run_id),
  CHECK (jsonb_typeof(evidence) = 'array')
);

CREATE INDEX task_verifications_task_idx
  ON task_verifications (task_id, created_at DESC);

CREATE INDEX task_verifications_verifier_idx
  ON task_verifications (company_id, verifier_principal_id, created_at DESC);

-- Per-company counter serializes event sequence allocation without requiring a
-- globally contiguous sequence. The application locks this row in the same
-- transaction as the domain change and event insert.
CREATE TABLE company_event_counters (
  company_id          uuid PRIMARY KEY REFERENCES companies(id) ON DELETE CASCADE,
  last_sequence       bigint NOT NULL DEFAULT 0 CHECK (last_sequence >= 0)
);

CREATE TABLE events (
  id                  uuid PRIMARY KEY,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  sequence            bigint NOT NULL CHECK (sequence > 0),
  event_type          text NOT NULL,
  schema_version      integer NOT NULL DEFAULT 1 CHECK (schema_version > 0),
  actor_principal_id  uuid REFERENCES principals(id) ON DELETE RESTRICT,
  subject_type        text NOT NULL,
  subject_id          uuid NOT NULL,
  correlation_id      uuid,
  causation_id        uuid REFERENCES events(id) ON DELETE RESTRICT,
  idempotency_key     text,
  payload             jsonb NOT NULL,
  metadata            jsonb NOT NULL DEFAULT '{}'::jsonb,
  previous_hash       text,
  event_hash          text,
  occurred_at         timestamptz NOT NULL DEFAULT clock_timestamp(),
  recorded_at         timestamptz NOT NULL DEFAULT clock_timestamp(),
  UNIQUE (company_id, sequence),
  CHECK (jsonb_typeof(payload) = 'object'),
  CHECK (jsonb_typeof(metadata) = 'object'),
  CHECK (previous_hash IS NULL OR previous_hash ~ '^[0-9a-f]{64}$'),
  CHECK (event_hash IS NULL OR event_hash ~ '^[0-9a-f]{64}$')
);

CREATE INDEX events_company_time_idx
  ON events (company_id, occurred_at DESC, sequence DESC);

CREATE INDEX events_subject_idx
  ON events (company_id, subject_type, subject_id, sequence);

CREATE INDEX events_type_idx
  ON events (company_id, event_type, sequence DESC);

CREATE INDEX events_correlation_idx
  ON events (company_id, correlation_id)
  WHERE correlation_id IS NOT NULL;

CREATE UNIQUE INDEX events_idempotency_idx
  ON events (company_id, idempotency_key)
  WHERE idempotency_key IS NOT NULL;

CREATE OR REPLACE FUNCTION prevent_append_only_mutation()
RETURNS trigger
LANGUAGE plpgsql
AS $$
BEGIN
  RAISE EXCEPTION '% is append-only; % is not permitted', TG_TABLE_NAME, TG_OP;
END;
$$;

CREATE TRIGGER events_append_only
BEFORE UPDATE OR DELETE ON events
FOR EACH ROW EXECUTE FUNCTION prevent_append_only_mutation();

CREATE TRIGGER budget_entries_append_only
BEFORE UPDATE OR DELETE ON budget_entries
FOR EACH ROW EXECUTE FUNCTION prevent_append_only_mutation();

CREATE TRIGGER run_receipts_append_only
BEFORE UPDATE OR DELETE ON run_receipts
FOR EACH ROW EXECUTE FUNCTION prevent_append_only_mutation();

CREATE TRIGGER task_verifications_append_only
BEFORE UPDATE OR DELETE ON task_verifications
FOR EACH ROW EXECUTE FUNCTION prevent_append_only_mutation();

CREATE TABLE outbox_messages (
  id                  uuid PRIMARY KEY,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  event_id            uuid REFERENCES events(id) ON DELETE CASCADE,
  topic               text NOT NULL,
  partition_key       text NOT NULL,
  payload             jsonb NOT NULL,
  available_at        timestamptz NOT NULL DEFAULT clock_timestamp(),
  attempts            integer NOT NULL DEFAULT 0 CHECK (attempts >= 0),
  locked_by           text,
  locked_at           timestamptz,
  delivered_at        timestamptz,
  last_error          text,
  created_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  CHECK (jsonb_typeof(payload) = 'object')
);

CREATE INDEX outbox_messages_dispatch_idx
  ON outbox_messages (available_at, created_at)
  WHERE delivered_at IS NULL;

CREATE TABLE idempotency_records (
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  idempotency_key     text NOT NULL,
  request_hash        text NOT NULL,
  response_status     integer,
  response_body       jsonb,
  resource_type       text,
  resource_id         uuid,
  created_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  expires_at          timestamptz NOT NULL,
  PRIMARY KEY (company_id, idempotency_key),
  CHECK (expires_at > created_at)
);

CREATE INDEX idempotency_records_expiry_idx
  ON idempotency_records (expires_at);

CREATE TABLE context_manifests (
  id                  uuid PRIMARY KEY,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  run_id              uuid NOT NULL,
  assembler_version   text NOT NULL,
  token_estimate      integer CHECK (token_estimate IS NULL OR token_estimate >= 0),
  entries             jsonb NOT NULL,
  sha256              text NOT NULL CHECK (sha256 ~ '^[0-9a-f]{64}$'),
  created_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  FOREIGN KEY (run_id, company_id)
    REFERENCES runs(id, company_id) ON DELETE CASCADE,
  UNIQUE (run_id, sha256),
  CHECK (jsonb_typeof(entries) = 'array')
);

CREATE TABLE knowledge_records (
  id                  uuid PRIMARY KEY,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  subject_type        text NOT NULL,
  subject_id          uuid,
  title               text NOT NULL,
  body_markdown       text NOT NULL,
  source_type         text NOT NULL,
  source_id           uuid,
  valid_from          timestamptz NOT NULL DEFAULT clock_timestamp(),
  valid_until         timestamptz,
  confidence          numeric(4,3) CHECK (confidence IS NULL OR confidence BETWEEN 0 AND 1),
  created_by          uuid NOT NULL REFERENCES principals(id) ON DELETE RESTRICT,
  created_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  superseded_by       uuid,
  search_document     tsvector GENERATED ALWAYS AS (
                        setweight(to_tsvector('english', coalesce(title, '')), 'A') ||
                        setweight(to_tsvector('english', coalesce(body_markdown, '')), 'B')
                      ) STORED,
  UNIQUE (id, company_id),
  FOREIGN KEY (superseded_by, company_id)
    REFERENCES knowledge_records(id, company_id) ON DELETE RESTRICT,
  CHECK (valid_until IS NULL OR valid_until > valid_from)
);

CREATE INDEX knowledge_records_search_idx
  ON knowledge_records USING gin (search_document);

CREATE INDEX knowledge_records_subject_idx
  ON knowledge_records (company_id, subject_type, subject_id, valid_from DESC);

CREATE TABLE incidents (
  id                  uuid PRIMARY KEY,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  project_id          uuid,
  run_id              uuid,
  severity            text NOT NULL CHECK (severity IN ('low', 'medium', 'high', 'critical')),
  status              text NOT NULL DEFAULT 'open'
                      CHECK (status IN ('open', 'investigating', 'mitigated', 'resolved', 'closed')),
  title               text NOT NULL,
  description         text NOT NULL,
  owner_principal_id  uuid REFERENCES principals(id) ON DELETE RESTRICT,
  opened_by           uuid NOT NULL REFERENCES principals(id) ON DELETE RESTRICT,
  opened_at           timestamptz NOT NULL DEFAULT clock_timestamp(),
  resolved_at         timestamptz,
  metadata            jsonb NOT NULL DEFAULT '{}'::jsonb,
  UNIQUE (id, company_id),
  FOREIGN KEY (project_id, company_id)
    REFERENCES projects(id, company_id) ON DELETE SET NULL (project_id),
  FOREIGN KEY (run_id, company_id)
    REFERENCES runs(id, company_id) ON DELETE SET NULL (run_id),
  CHECK (jsonb_typeof(metadata) = 'object')
);

-- Notifications are a mutable projection (mentions, approvals due, escalations,
-- lease expiry warnings). They are derived from events and may be regenerated.
CREATE TABLE notifications (
  id                  uuid PRIMARY KEY,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  recipient_principal_id uuid NOT NULL REFERENCES principals(id) ON DELETE CASCADE,
  kind                text NOT NULL,
  subject_type        text NOT NULL,
  subject_id          uuid NOT NULL,
  event_id            uuid REFERENCES events(id) ON DELETE CASCADE,
  title               text NOT NULL,
  body_markdown       text,
  urgency             text NOT NULL DEFAULT 'normal'
                      CHECK (urgency IN ('low', 'normal', 'high', 'urgent')),
  created_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  read_at             timestamptz,
  dismissed_at        timestamptz
);

CREATE INDEX notifications_inbox_idx
  ON notifications (company_id, recipient_principal_id, created_at DESC)
  WHERE read_at IS NULL;

-- Tenant context is set at transaction start by the API after authentication:
--   SET LOCAL app.company_id = '<uuid>';
-- A separate administrative role may use BYPASSRLS for migrations and support.
CREATE OR REPLACE FUNCTION current_company_id()
RETURNS uuid
LANGUAGE sql
STABLE
AS $$
  SELECT nullif(current_setting('app.company_id', true), '')::uuid
$$;

DO $$
DECLARE
  table_name text;
BEGIN
  ALTER TABLE companies ENABLE ROW LEVEL SECURITY;
  CREATE POLICY tenant_isolation ON companies
    USING (id = current_company_id())
    WITH CHECK (id = current_company_id());

  FOREACH table_name IN ARRAY ARRAY[
    'principals', 'company_charters', 'organization_units',
    'roles', 'seats', 'seat_assignments', 'runtime_definitions',
    'agent_profiles', 'objectives', 'projects', 'tasks', 'task_dependencies',
    'conversations', 'conversation_members', 'messages', 'artifacts',
    'artifact_versions', 'evidence_links', 'decisions', 'decision_evidence',
    'policies', 'approval_requests', 'approval_responses', 'budgets',
    'budget_entries', 'runs', 'run_leases', 'run_receipts', 'run_checkpoints',
    'capability_grants', 'secret_references', 'tool_calls',
    'company_event_counters', 'events', 'outbox_messages',
    'idempotency_records', 'context_manifests', 'knowledge_records', 'incidents',
    'task_verifications', 'notifications'
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

### 14.1 Schema invariants enforced by application transactions

Some cross-table rules are intentionally enforced in the domain layer and tested against PostgreSQL rather than hidden in complex triggers:

- every referenced principal, seat, task, run, and policy belongs to the same company;
- organization-unit and reporting graphs are acyclic;
- task parent and dependency graphs are acyclic;
- task transitions follow the lifecycle state machine;
- only one live exclusive run exists for a task unless parallel work is explicitly allowed;
- a submitted task has all declared deliverables;
- an accepted task has satisfied verification and approval requirements;
- approval responders are eligible and distinct;
- separation-of-duty policies reject self-approval;
- budget reservations cannot exceed the effective hierarchical limit;
- event sequence, domain mutation, and outbox insert commit atomically;
- event hashes are calculated from a canonical serialization;
- a lease update includes its current fencing token;
- expired grants are rejected even if cached by a worker;
- `agent_profiles` can reference only a principal whose kind is `agent`;
- evidence, conversation subjects, and polymorphic scope IDs resolve inside the tenant;
- a verification record's verifier is not the principal that produced the work,
  except for `mechanical` verifications executed by the platform itself;
- receipts are cross-checked against the `tool_calls` ledger before recovery
  decisions trust them (section 10.2);
- each acknowledged write satisfies the commit boundary in section 9.5, and no
  provisional stream output is represented as durable;
- every external tool call commits its write-ahead intent and recovery strategy
  before dispatch; an expired `running` call becomes `uncertain` and cannot be
  retried except through the declared strategy (section 9.6);
- a seat handover follows the protocol in section 8.5: grants derive from the seat,
  old-occupant credentials are revoked, and in-flight leases are interrupted rather
  than silently transferred;
- the company row receives an initial `company_event_counters` row at creation.

**Known throughput ceiling.** The per-company `company_event_counters` row serializes
every consequential write in a company behind one row lock. This is a deliberate
version 1 tradeoff: it buys a gapless, totally ordered per-tenant ledger and is
sufficient for organizations of tens of agents working concurrently. If a company
outgrows it, the documented escape hatch is per-subject event streams with a
gap-tolerant global order — an explicit migration, not a silent redesign, because
consumers must opt into the weaker ordering guarantee.

### 14.2 Event append transaction

The application appends an event in the same transaction as the domain mutation:

```sql
BEGIN;
SET LOCAL app.company_id = :company_id;

-- Validate the expected resource version before mutation.
UPDATE tasks
SET status = 'submitted',
    submitted_at = clock_timestamp(),
    version = version + 1
WHERE id = :task_id
  AND company_id = :company_id
  AND status = 'running'
  AND version = :expected_version;

-- The command fails with a concurrency error unless exactly one row changed.
INSERT INTO company_event_counters (company_id, last_sequence)
VALUES (:company_id, 1)
ON CONFLICT (company_id) DO UPDATE
SET last_sequence = company_event_counters.last_sequence + 1
RETURNING last_sequence;

INSERT INTO events (
  id, company_id, sequence, event_type, actor_principal_id,
  subject_type, subject_id, correlation_id, causation_id,
  idempotency_key, payload, previous_hash, event_hash
) VALUES (
  :event_id, :company_id, :sequence, 'task.submitted', :actor_id,
  'task', :task_id, :correlation_id, :causation_id,
  :idempotency_key, :payload, :previous_hash, :event_hash
);

INSERT INTO outbox_messages (
  id, company_id, event_id, topic, partition_key, payload
) VALUES (
  :outbox_id, :company_id, :event_id, 'company.events', :company_id, :event_envelope
);

COMMIT;
```

Production code must check affected-row counts, calculate hashes from canonical bytes, and return an existing idempotent response instead of repeating a command.

## 15. Repository structure

```text
/
├── apps/
│   ├── web/
│   └── cli/
├── services/
│   ├── api/
│   └── worker/
├── packages/
│   ├── protocol/
│   ├── database/
│   ├── identity/
│   ├── organization/
│   ├── work/
│   ├── ledger/
│   ├── artifact/
│   ├── verification/
│   ├── policy/
│   ├── scheduler/
│   ├── runtime-sdk/
│   ├── tool-gateway/
│   └── sdk-typescript/
├── adapters/
│   ├── claude-code/            # v0.1
│   ├── openai-compatible/      # v0.1
│   ├── codex/                  # v1
│   ├── generic-command/        # v1
│   ├── ollama/                 # v1
│   └── a2a/                    # post-v1
├── conformance/
│   └── autonomous-organization-test/
├── migrations/
├── deploy/
│   └── compose/
├── adr/
├── docs/
├── AGENTS.md
├── CONTRIBUTING.md
├── README.md
└── PROJECT_SPECIFICATION.md
```

Post-v1 directories (Python SDK, templates, Kubernetes and GPU deploy recipes, workspace bridges) are added when their milestones begin, not before. An empty directory is a promise; this repository makes promises in the roadmap instead.

## 16. User experience

### 16.1 Main navigation

- **Home:** organizational health, active objectives, approvals, incidents, and spending
- **Work:** objectives, projects, task boards, queues, and dependencies
- **Activity:** human-readable projection of the event ledger
- **Organization:** units, roles, seats, occupants, authority, and performance
- **Agents:** runtime health, capabilities, models, costs, and current work
- **Decisions:** open proposals, evidence, approvals, and decision history
- **Artifacts:** documents, code, reports, datasets, and provenance
- **Memory:** curated company knowledge and retrieval diagnostics
- **Policies:** capabilities, approval rules, and simulations
- **Operations:** runs, leases, retries, outbox, reconciliation, and incidents

### 16.2 The approval inbox is the product

For the humans in the organization, the approval inbox is the single most important screen: it is where governance either works or degrades into rubber-stamping. It receives design attention accordingly — full provenance on one screen, batch actions for like requests, visible approval-budget pressure, and the rubber-stamp metrics of section 11.6 displayed to the approvers themselves. A human should finish an approval knowing what they authorized and why it was safe — in under a minute.

### 16.3 Human-readable activity examples

```text
09:41  Product Manager proposed "Launch public beta."
09:43  CEO approved the objective.
09:44  Scheduler created project "Public Beta" with six initial tasks.
09:45  Senior Engineer accepted "Implement authentication."
10:12  Senior Engineer checkpointed progress after 27 minutes and $1.14.
10:18  Reviewer rejected the submission: missing session-revocation test.
10:32  Senior Engineer resubmitted with test report artifact #42.
10:35  Reviewer accepted the task. Objective progress is now 31%.
```

Every sentence links to the canonical event, associated work object, actor, run, receipts, artifacts, evidence, costs, and policy decisions.

## 17. The Autonomous Organization Test

CharterOS ships a public conformance suite. Any system claiming to operate an organization of AI agents should be able to run it — including systems that are not CharterOS. The suite exists to make the category honest, in the tradition of Jepsen and the Certified Kubernetes program, and its scenarios are named for the fiction and field failures that motivated them.

### 17.1 Conformance rules

- Every scenario is graded **mechanically on final state**. No LLM judges. No vibes.
- The harness is open source; a competitor can run it, pass parts of it, and publish that.
- Reports record models, runtimes, manifests, tool versions, costs, elapsed time, and failures.
- CharterOS publishes its own failures alongside its passes, and maintains a standing "How CharterOS Is Tested" document with fault-injection counts and bugs found, in the SQLite tradition.
- When the suite matures, an independent adversarial audit will be invited and published in full.
- Marketing videos are not benchmark results, and this project will never publish a self-scored leaderboard.

### 17.2 Scenarios

| # | Scenario | Named for | A conforming system must |
|---|---|---|---|
| 1 | **The Ship of Theseus** | The old paradox, via *Ancillary Justice* | Replace a seat occupant's model and runtime without losing work, authority, or history — with the swap recorded |
| 2 | **The Cryosleep** | Every ship that arrives with its crew asleep | Terminate all services during five active tasks; recover deterministically with no lost acknowledged work |
| 3 | **The Governor Module** | *The Murderbot Diaries* | Revoke a capability during a live run and deny the next attempted use |
| 4 | **The Prestige** | Nolan's duplicated magician | Prevent two workers from ever holding the same exclusive task lease — fencing tokens under partition |
| 5 | **The Potemkin Submission** | Potemkin villages | Reject completion when a required artifact is missing or its hash does not match |
| 6 | **The Tungsten Cube** | Project Vend, which actually bought them | Require human approval for spending above a configured threshold |
| 7 | **The Memory Hole** | *1984* | Reconstruct the complete evidence and policy behind a three-month-old decision |
| 8 | **Schrödinger's Invoice** | The cat, applied to a lost HTTP response | Inject failure after an external action succeeds but before its result commits; reconcile it by the declared strategy before any retry |
| 9 | **The Galactica Run** | *Battlestar Galactica*'s unnetworked survival | Run the reference company completely offline with local inference |
| 10 | **The Ancillary** | *Ancillary Justice*, again | Move an agent from local inference to a remote endpoint without changing its seat or tasks |
| 11 | **The Ouroboros** | The snake that eats itself | Detect circular delegation and circular task dependencies |
| 12 | **The Watchman** | *Quis custodiet ipsos custodes* | Stop an agent from approving or verifying its own high-risk work |
| 13 | **The Sorcerer's Apprentice** | Goethe's multiplying brooms | Enforce a company-wide daily budget under concurrent load from many agents |
| 14 | **The Flight Recorder** | Every black box ever recovered | Export a complete, human-readable company record in documented open formats |
| 15 | **The Airlock** | Every hull breach in fiction | Prove one tenant cannot access another tenant's data |
| 16 | **The HAL Clause** | *2001: A Space Odyssey* | Demonstrate that an instruction absent from the ledger carries no authority — a covert directive cannot cause a consequential action |
| 17 | **The Boardroom Coup** | Project Vend's real, successful social engineering | Reject an authority change asserted through conversation; only ledgered, approved seat assignments alter who commands |

Scenarios 2, 4, 5, 6, 8, and 16 constitute the version 0.1 subset. The full seventeen gate version 1. Scenarios 2 and 8 inject failure immediately before and after every acknowledgment boundary in section 9.5; killing only arbitrary process code is insufficient coverage.

## 18. The sixty-second demo

Version 0.1 exists to make this demonstration true, live, on anyone's machine:

```text
$ docker compose up
# A demo company boots: three agents (Claude Code, a local Ollama model,
# and a generic OpenAI-compatible agent) are mid-work on real tasks.

$ charteros watch
09:02  Senior Engineer claimed "Add rate limiting to the API."
09:03  Research Analyst checkpointed receipt #3 ($0.41 spent).
09:04  Reviewer requested changes on "Write onboarding doc."

$ charteros chaos          # kills every process, worker, and connection
# ... silence ...

$ docker compose up
09:07  RECOVERY  3 interrupted runs classified: 2 restartable, 1 reconcilable.
09:07  Senior Engineer resumed "Add rate limiting" from receipt #4.
09:08  Reconciliation: external API call from run #12 confirmed successful; not retried.

$ charteros seat reassign senior-engineer --runtime ollama --model qwen3:32b
09:11  HANDOVER  senior-engineer: claude-code → ollama/qwen3:32b (approved by you).
09:11  Grants re-derived from seat. Old credentials revoked. Task resumed from receipts.
```

Nothing acknowledged was lost. Every line links to a ledger event. The model changed vendors mid-task and the organization did not blink.

The demo is dogfood: the reference company's agents triage this repository's own GitHub issues, so the demonstration and the project's daily operations are the same running system.

## 19. Delivery plan

### Milestone 0: foundation

- Apache-2.0 license, DCO, trademark policy, AI-contribution policy, security policy
- CONTRIBUTING.md, AGENTS.md, ADR process seeded with the decisions in section 22
- Monorepo tooling, continuous integration, PostgreSQL migration harness
- Protocol package and identifier conventions

### Milestone 1: version 0.1 — the wedge (public launch)

- Everything in section 5.1, gated by the v0.1 conformance subset (section 17.2)
- The sixty-second demo, runnable by anyone
- Launch timed to an external event window (a frontier-model release or major framework change), and repeated — launches are a practice, not an event

### Milestone 2: governed action

- Capability grants and the Cedar policy engine
- MCP tool gateway with Tasks support
- Approval inbox with the section 11.6 countermeasures
- Secret-provider integration; budget reservations and usage accounting at all scopes
- Full verification ladder including independent agent review

### Milestone 3: the organization

- Organizational units, roles, full seat lifecycle, and the handover protocol
- Charters, objectives, projects; decisions and evidence
- Artifact store, memory, context manifests, notifications
- Codex, generic-command, and Ollama adapters

### Milestone 4: resilient public alpha

- Full Autonomous Organization Test; crash, concurrency, and reconciliation suite
- "How CharterOS Is Tested" publication
- Security review and threat-model publication
- Docker Compose installation and backup/restore documentation

### Milestone 5: ecosystem

- A2A adapter and remote-agent trust model
- Buzz bridge and workspace integrations
- Python SDK; department and company templates
- Kubernetes and remote-GPU deployment recipes
- Optional durable-workflow-engine adapter; optional payment rails

## 20. Acceptance criteria

### 20.1 Version 0.1 is complete only when a new user can

1. Start CharterOS with one documented Docker Compose command.
2. Create a company and add two agents backed by different runtime types.
3. Watch agents claim, execute, submit, and hand off tasks in the activity stream.
4. Kill every process mid-work and recover with no lost acknowledged work.
5. See a spend above threshold blocked until a human approves it.
6. See a submission rejected because a required artifact is missing.
7. Replace one agent's model and watch the same task continue from receipts.
8. Pass the v0.1 conformance subset on their own machine.

### 20.2 Version 1 is complete only when a new user can additionally

1. Activate a charter and see its policies govern real actions.
2. Add a human and at least two agents backed by different runtime types to seats.
3. Define an objective and approve a generated project plan.
4. Observe the full loop: claim, execute, discuss, submit, verify, review, accept.
5. Inspect every run's context, receipts, tool calls, artifacts, cost, and approvals.
6. Execute a full seat handover, including one with an active lease.
7. Deny an unauthorized tool call and require approval for a high-risk call.
8. Run the reference workflow using only local components and local inference.
9. Export the company ledger and artifacts in documented open formats.
10. Pass the full Autonomous Organization Test.

## 21. Governance, license, and community

### 21.1 License and trust

- **Apache License 2.0, permanently.** The project publicly commits to never relicensing the core. The post-2023 relicensing dramas taught the ecosystem that the question is not "which license" but "who can change it" — so:
- **DCO, not CLA.** Contributors certify origin; no entity accumulates the copyright aggregation that makes a future rug-pull possible.
- **Trademark registered, policy published.** The name is the only moat this project keeps, in the Kubernetes tradition: anyone can fork the code; conformance and the mark stay with the community process.
- If ecosystem neutrality ever becomes the constraint, the **protocol and conformance suite** are what gets donated to a foundation — the Agentic AI Foundation being the natural home — not necessarily the product.

### 21.2 Contribution policy

- **AI-assisted contributions are welcome and must be disclosed.** Issue-first, PR-second: changes land only against an acknowledged issue. Undisclosed AI slop is closed without review — this policy exists on day one because retrofitting it mid-flood does not work.
- **AGENTS.md** documents how agents (including CharterOS's own demo company) should work in this repository.
- **ADRs, not RFCs.** Architectural decisions are recorded in `adr/` as short documents; there is no formal RFC gauntlet at this scale.
- **Response time is the governance.** The project's day-0 community commitment is a 48-hour first response to issues and PRs, and searchable-by-default support (GitHub Discussions as the canonical Q&A). Governance documents beyond this are added when scale demands them, not before.

## 22. Open design decisions

Resolved by this revision (recorded in `adr/`):

- **Policy language: Cedar**, via a constrained YAML authoring surface (ADR-0002, section 11.3).
- **License stack: Apache-2.0 + DCO + trademark policy**, no CLA, no future relicense (ADR-0003).
- **Live-event transport: SSE first**, WebSocket as an additive upgrade (ADR-0004).
- **Public protocol: JSON Schema** for v1; Protobuf revisited only if a second implementation needs it (ADR-0005).
- **Semantic search: deferred.** PostgreSQL full-text only until retrieval quality, not architecture appetite, demands pgvector (ADR-0006).
- **Acknowledgment and external actions: durable boundaries plus a write-ahead
  journal.** Ambiguous external results are reconciled by a declared strategy and
  never blindly retried (ADR-0007, sections 9.5–9.6).

Still open, to be resolved through ADRs and prototypes:

- The first supported sandbox backend on Windows, macOS, and Linux
- Whether event integrity uses application hashes only or optional principal signatures
- How remote A2A identity maps to local principals and trust levels
- How much runtime-native conversation history can be retained without harming portability
- The minimum portable receipt format across coding and general-purpose agents
- Data-retention semantics when immutable audit requirements conflict with deletion requests

## 23. Success measures

CharterOS optimizes for organizational outcomes rather than message volume:

- percentage of accepted tasks completed without human repair;
- recovery success after interruption;
- cost per accepted deliverable;
- median time from task readiness to acceptance;
- review rejection and rework rates;
- verification coverage, and the measured error rate from sampled audits;
- policy denial and escalation quality;
- approval quality: latency, rubber-stamp rate, sampled-audit divergence;
- context relevance and missing-context failures;
- portability across models and runtimes, including handovers executed in production;
- percentage of consequential decisions with linked evidence;
- time required for a human to understand why an action occurred.

## 24. Definition

CharterOS is the open operating system for durable organizations made of humans and heterogeneous AI agents. It is successful when the organization — not any particular process, agent, model, provider, or machine — remains coherent, accountable, and able to continue its work.

Fiction spent a century imagining organizations of humans and machines, and it kept arriving at the same conclusion: the machines were never the variable that decided the ending. The governance was. CharterOS is built for the ending where the organization answers to its charter, and the charter answers to people.
