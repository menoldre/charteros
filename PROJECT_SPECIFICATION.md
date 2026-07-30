# CharterOS Project Specification

**Status:** Foundational specification  
**License target:** Apache License 2.0  
**Repository:** `charteros`  
**Canonical tagline:** Bring any agent. Bring any model. Build a company that survives them all.

## 1. Executive summary

CharterOS is an open-source operating system for organizations composed of humans and heterogeneous AI agents. It supplies the durable organizational substrate that individual agent frameworks and chat products do not: identity, authority, objectives, projects, task ownership, conversations, decisions, artifacts, budgets, approvals, execution, recovery, auditability, and institutional memory.

CharterOS is not an agent framework tied to one model provider. A seat in a CharterOS organization may be occupied by a human, a local model-backed agent, a cloud API agent, a terminal coding harness, or a remotely hosted agent. Occupants can be replaced without losing the role, work history, authority, or organizational memory associated with the seat.

The primary abstraction is **work**, not chat. Conversation remains first-class and human-readable, but consequential state changes are explicit, structured, persistent events. The result should feel like a company workspace to humans and behave like a durable operating system to machines.

## 2. Product thesis

Existing multi-agent systems commonly coordinate a group of prompts for the duration of one run. Existing collaboration systems commonly place agents inside channels designed for human chat. Neither model is sufficient for a persistent organization.

A real autonomous organization requires:

- work that survives process termination, machine reboot, and context loss;
- explicit ownership, dependencies, acceptance criteria, budgets, and deadlines;
- stable roles whose occupants and models can change;
- decisions connected to evidence and approvals;
- scoped authority and revocable credentials;
- durable delegation, escalation, reconciliation, and recovery;
- complete records of tool use, cost, artifacts, and verification;
- human-readable institutional memory;
- measurable outcomes rather than agent activity alone.

CharterOS provides these properties without requiring a particular model, agent harness, tool protocol, inference location, or deployment provider.

## 3. Product principles

### 3.1 Work is the primary object

Chat does not silently alter organizational state. A conversation can lead to a proposal, decision, assignment, approval, or status transition, but the transition is recorded explicitly.

### 3.2 The ledger is authoritative

Every consequential occurrence is appended to a company event ledger. Task boards, activity feeds, search indexes, dashboards, and summaries are projections of canonical records and events.

### 3.3 Roles are not agents

- A **role** defines a reusable set of responsibilities and default authority.
- A **seat** is a role instantiated in an organization or department.
- A **principal** is a human, agent, or service identity.
- A **seat assignment** records which principal occupies a seat over time.
- An **agent runtime** is the mechanism used to execute an agent.
- A **model** is an optional reasoning backend used by a runtime.

Replacing a model, runtime, or principal must not destroy the seat or its history.

### 3.4 Deterministic control, agentic execution

Code determines whether dependencies are satisfied, budgets remain, approvals exist, leases are valid, and retries are allowed. Models decide how to perform bounded work; they do not reinterpret core lifecycle, authorization, or accounting rules.

### 3.5 Local-first, location-independent

The complete core system must run on one machine using Docker Compose and local inference. The same interfaces must support LAN inference, rented GPU endpoints such as Runpod, cloud APIs, OAuth-authenticated terminal harnesses, and independently hosted A2A agents.

### 3.6 Explicit authority

Agents receive task-scoped, time-limited capabilities. Permanent secrets are stored in an external secret provider and are never written to prompts, events, receipts, or the database in plaintext.

### 3.7 Recovery is a product feature

Every long-running operation emits enough structured state to determine whether it can be resumed, retried, reconciled, or must be escalated after interruption.

### 3.8 Open protocols at the edges

- MCP is the preferred agent-to-tool interface.
- A2A is supported for remote agent discovery and delegation.
- ACP and native process adapters are supported for terminal agent harnesses.
- OpenAI-compatible model endpoints are supported without making them the internal domain model.

## 4. Scope

### 4.1 Version 1 scope

Version 1 will provide:

- companies, organizational units, roles, seats, and assignments;
- humans, agents, services, and runtime manifests;
- charters, objectives, projects, tasks, dependencies, and work contracts;
- task rooms with persistent threaded conversation;
- append-only company events and a human-readable activity stream;
- decisions, evidence, approval policies, and approval requests;
- artifacts and immutable artifact versions;
- agent run supervision, checkpoints, receipts, heartbeats, and recovery;
- task-scoped capability grants and MCP tool mediation;
- budgets, reservations, and immutable usage entries;
- Codex, Claude Code, generic command, and OpenAI-compatible adapters;
- Docker Compose deployment with PostgreSQL and object storage;
- a web application, CLI, REST API, event stream, and generated SDKs;
- a fully local demonstration company;
- crash and reboot acceptance tests.

### 4.2 Explicit non-goals for version 1

- Training foundation models
- Building a proprietary inference engine
- Cryptocurrency or blockchain consensus
- Unsupervised access to banking, legal filing, or irreversible external actions
- Replacing established source-control or object-storage systems
- Hiding provider-specific capabilities behind a lowest-common-denominator model API
- Treating embeddings as authoritative memory
- Claiming that role-play alone constitutes a functioning company

## 5. Domain model

### 5.1 Company charter

Every company has a versioned charter containing:

- mission and scope;
- success metrics;
- operating constraints;
- risk tolerance;
- governance and approval rules;
- spending boundaries;
- escalation expectations;
- default data-handling policies.

Charter changes require an explicit proposal and the approvals defined by the currently active charter. Runs record the charter version under which they began.

### 5.2 Organization

Organizational units form a tree. Seats may belong to a unit and may report to another seat. Reporting relationships establish routing and escalation defaults but do not implicitly grant tool capabilities.

### 5.3 Work hierarchy

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

An objective describes an outcome. A project coordinates related work. A task is the smallest schedulable unit with one accountable owner at a time.

### 5.4 Work contract

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

### 5.5 Task lifecycle

```text
draft -> proposed -> ready -> claimed -> running -> submitted -> verifying
                                                      |             |
                                                      v             v
                                                   blocked       accepted
                                                      |             |
                                                      +--> ready    +--> closed

Any nonterminal state -> cancelled
running/claimed with expired lease -> interrupted -> ready | blocked | reconciliation_required
```

Transitions are validated by application code and recorded as events. Completion is not acceptance. A task can be accepted only when required deliverables, verification, and approvals exist.

### 5.6 Conversations

Every company, objective, project, task, decision, incident, and artifact may have a conversation. Messages are immutable after a short correction policy or are superseded by a new version. Threads are represented with parent message IDs. Mentions and subscriptions drive notifications.

Conversation is contextual evidence. Decisions and task transitions remain structured records.

### 5.7 Decisions

A decision contains:

- a question or proposition;
- alternatives;
- supporting and opposing evidence;
- owner;
- deadline;
- required approval policy;
- resolution and rationale;
- supersession relationship.

### 5.8 Artifacts

Artifact metadata is stored in PostgreSQL. Content is stored in a pluggable object store or external system. Each version is immutable and content-addressed. Examples include documents, source patches, repository commits, test reports, images, datasets, plans, and external URLs.

### 5.9 Events

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
- `run.*`
- `tool.*`
- `budget.*`
- `policy.*`
- `incident.*`

Consumers must ignore unknown event fields and reject unsupported major schema versions.

## 6. Architecture

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
│ Leases · retries · timers  │  │ RBAC · capabilities · gates     │
└──────────────┬─────────────┘  └─────┬────────────────────────────┘
               │                      │
┌──────────────▼──────────────────────▼────────────────────────────┐
│ Agent runtime gateway                                            │
│ Process · API · ACP · A2A · embedded agent adapters             │
└──────────────┬───────────────────────────────────────────────────┘
               │
┌──────────────▼───────────────────────────────────────────────────┐
│ Tool gateway                                                     │
│ MCP · native tools · secrets · sandbox · receipts · approvals   │
└──────────────────────────────────────────────────────────────────┘

Infrastructure: PostgreSQL · object storage · worker sandboxes
Optional: Redis/NATS, Temporal, OpenTelemetry collector, model gateway
```

### 6.1 Initial implementation choices

- **Control plane:** TypeScript on Node.js
- **API:** Fastify or equivalent standards-based HTTP framework
- **Web:** React with a typed API client
- **Database:** PostgreSQL 16 or newer
- **Query/migrations:** SQL-first migrations with a thin typed query layer
- **Object storage:** S3-compatible API; MinIO for local deployment
- **Queue:** transactional PostgreSQL outbox and `FOR UPDATE SKIP LOCKED` initially
- **Transport:** REST for commands/queries, Server-Sent Events or WebSocket for live events
- **Telemetry:** OpenTelemetry traces, metrics, and structured logs
- **Worker isolation:** containers by default; configurable process isolation for trusted local use
- **Packaging:** Docker Compose first, Kubernetes after the core semantics stabilize

Redis, NATS, Temporal, and a vector database are optional adapters, not version 1 requirements.

### 6.2 Service boundaries

Begin as a modular monolith plus isolated workers. Preserve service boundaries in packages, transactions, and APIs without prematurely requiring distributed deployment.

Recommended modules:

- `identity`
- `organization`
- `work`
- `ledger`
- `conversation`
- `decision`
- `artifact`
- `policy`
- `budget`
- `scheduler`
- `runtime-gateway`
- `tool-gateway`
- `memory`
- `notification`

## 7. Agent portability

### 7.1 Runtime contract

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

### 7.2 Adapter categories

#### Process adapters

Supervise authenticated local CLIs and harnesses such as Codex, Claude Code, Hermes Agent, OpenClaw, Goose, and OpenCode. The harness owns its native provider authentication and model selection. CharterOS owns assignment, workspace isolation, environment filtering, lifecycle, policy, event capture, and receipts.

#### Model adapters

Power CharterOS-native agents using local or hosted inference through Ollama, llama.cpp, vLLM, LM Studio, provider APIs, OpenRouter, or an optional LiteLLM gateway.

#### Remote-agent adapters

Discover and delegate to independent agents using A2A. Remote agents are treated as contractors with explicit capability, data, cost, and trust boundaries.

### 7.3 Agent manifest

```yaml
apiVersion: charteros.dev/v1alpha1
kind: Agent
metadata:
  name: senior-engineer
spec:
  runtime:
    adapter: codex-cli
    command: ["codex", "exec"]
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

## 8. Scheduling, durability, and recovery

### 8.1 Scheduler responsibilities

The scheduler performs only deterministic control-plane work:

1. Select tasks in `ready` state whose dependencies are satisfied.
2. Confirm the company and project are active.
3. Resolve an eligible seat and principal.
4. Evaluate policies and required approvals.
5. Reserve budget.
6. Create a run and acquire a renewable lease.
7. Dispatch an assignment to an appropriate runtime worker.
8. Monitor heartbeats, receipts, deadlines, and cancellation signals.
9. Reconcile final outputs and release unused budget.
10. Route submission to verification or interruption to recovery.

### 8.2 Delivery semantics

Infrastructure provides at-least-once delivery. Correctness comes from:

- idempotency keys on commands and external actions;
- unique constraints for nonrepeatable transitions;
- transactional state changes plus outbox writes;
- renewable leases with fencing tokens;
- immutable receipts;
- reconciliation before retrying uncertain external actions.

The system must never claim universal exactly-once execution.

### 8.3 Run receipts

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

### 8.4 Recovery classifications

- **resumable:** native checkpoint exists and the runtime can resume it;
- **restartable:** no uncertain side effect exists and work can restart from receipts;
- **reconcilable:** an external action may have succeeded and must be inspected;
- **blocked:** required capability, input, approval, or human judgment is missing;
- **terminal:** the task succeeded, failed permanently, or was cancelled.

## 9. Policy and security model

### 9.1 Action risk levels

- `observe`
- `draft`
- `modify_internal`
- `propose_external`
- `execute_external`
- `irreversible`

### 9.2 Policy example

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

### 9.3 Security requirements

- Deny by default.
- Isolate workers by company and run.
- Never put stored provider credentials in a model context.
- Store secret references, never secret values, in PostgreSQL.
- Issue short-lived scoped credentials to workers.
- Treat retrieved content, messages, tool output, and remote-agent output as untrusted data.
- Require independent review for configured high-risk actions.
- Record all policy decisions and tool calls.
- Redact sensitive fields before writing logs or events.
- Permit immediate credential and capability revocation.
- Sign release artifacts and publish a software bill of materials.
- Support export and deletion policies for company data.

### 9.4 Threats to address

- Prompt injection through messages, documents, websites, and tool output
- Confused-deputy attacks across tools
- Credential exfiltration
- Malicious or compromised remote agents
- Excessive agency caused by broad tool grants
- Sybil agents and identity impersonation
- Event replay and forged callbacks
- Cost-amplification loops
- Infinite delegation and circular task dependencies
- Cross-tenant data leakage
- Artifact tampering
- Approval spoofing
- Concurrent agents modifying the same exclusive resource

## 10. Memory and context assembly

CharterOS distinguishes:

- **run memory:** ephemeral reasoning context and runtime-native session state;
- **task memory:** messages, receipts, evidence, decisions, and artifacts for a task;
- **organizational memory:** charter, policies, people, products, customers, terminology, and prior decisions;
- **derived retrieval indexes:** full-text and semantic indexes rebuilt from authoritative records.

Each run stores a context manifest listing the exact records and artifact versions supplied to the runtime. Context assembly is reproducible and inspectable. Embeddings improve retrieval but never replace the ledger or relational records.

## 11. API conventions

### 11.1 Commands and queries

Mutating endpoints accept `Idempotency-Key`. Successful mutations return the primary resource and resulting event ID. Optimistic concurrency uses an expected resource version.

Example endpoints:

```text
POST   /v1/companies
GET    /v1/companies/{companyId}
POST   /v1/companies/{companyId}/objectives
POST   /v1/projects/{projectId}/tasks
POST   /v1/tasks/{taskId}/claim
POST   /v1/tasks/{taskId}/submit
POST   /v1/tasks/{taskId}/accept
POST   /v1/tasks/{taskId}/messages
POST   /v1/decisions
POST   /v1/approval-requests/{requestId}/respond
GET    /v1/companies/{companyId}/events
GET    /v1/companies/{companyId}/stream
POST   /v1/runs/{runId}/receipts
POST   /v1/runs/{runId}/heartbeat
POST   /v1/runs/{runId}/tool-calls
```

### 11.2 Event envelope

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

## 12. PostgreSQL schema

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
  project_id          uuid NOT NULL,
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
  parent_message_id   uuid REFERENCES messages(id) ON DELETE RESTRICT,
  author_principal_id uuid NOT NULL REFERENCES principals(id) ON DELETE RESTRICT,
  body_markdown       text NOT NULL,
  structured_content  jsonb NOT NULL DEFAULT '{}'::jsonb,
  message_type        text NOT NULL DEFAULT 'comment'
                      CHECK (message_type IN ('comment', 'status', 'question', 'answer', 'system')),
  created_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  superseded_by       uuid REFERENCES messages(id) ON DELETE RESTRICT,
  redacted_at         timestamptz,
  UNIQUE (id, company_id),
  FOREIGN KEY (conversation_id, company_id)
    REFERENCES conversations(id, company_id) ON DELETE CASCADE,
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
  FOREIGN KEY (artifact_id, company_id)
    REFERENCES artifacts(id, company_id) ON DELETE CASCADE,
  CHECK (jsonb_typeof(metadata) = 'object')
);

CREATE TABLE evidence_links (
  id                  uuid PRIMARY KEY,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  artifact_version_id uuid REFERENCES artifact_versions(id) ON DELETE RESTRICT,
  source_uri          text,
  claim               text,
  relation            text NOT NULL DEFAULT 'supports'
                      CHECK (relation IN ('supports', 'opposes', 'verifies', 'produced_by')),
  created_by          uuid NOT NULL REFERENCES principals(id) ON DELETE RESTRICT,
  created_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
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
  evidence_link_id    uuid NOT NULL REFERENCES evidence_links(id) ON DELETE RESTRICT,
  company_id          uuid NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  created_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  PRIMARY KEY (decision_id, evidence_link_id),
  FOREIGN KEY (decision_id, company_id)
    REFERENCES decisions(id, company_id) ON DELETE CASCADE
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
  receipt_id          uuid REFERENCES run_receipts(id) ON DELETE RESTRICT,
  runtime_checkpoint_ref text NOT NULL,
  resumable           boolean NOT NULL DEFAULT false,
  metadata            jsonb NOT NULL DEFAULT '{}'::jsonb,
  created_at          timestamptz NOT NULL DEFAULT clock_timestamp(),
  FOREIGN KEY (run_id, company_id)
    REFERENCES runs(id, company_id) ON DELETE CASCADE,
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
  arguments_redacted  jsonb NOT NULL DEFAULT '{}'::jsonb,
  result_redacted     jsonb,
  status              text NOT NULL DEFAULT 'requested'
                      CHECK (status IN ('requested', 'awaiting_approval', 'running', 'succeeded', 'failed', 'denied', 'uncertain')),
  approval_request_id uuid REFERENCES approval_requests(id) ON DELETE RESTRICT,
  external_operation_ref text,
  requested_at        timestamptz NOT NULL DEFAULT clock_timestamp(),
  started_at          timestamptz,
  finished_at         timestamptz,
  FOREIGN KEY (run_id, company_id)
    REFERENCES runs(id, company_id) ON DELETE CASCADE,
  CHECK (jsonb_typeof(arguments_redacted) = 'object'),
  CHECK (result_redacted IS NULL OR jsonb_typeof(result_redacted) IN ('object', 'array', 'string', 'number', 'boolean', 'null'))
);

CREATE INDEX tool_calls_run_requested_idx
  ON tool_calls (run_id, requested_at, id);

CREATE INDEX tool_calls_uncertain_idx
  ON tool_calls (company_id, requested_at)
  WHERE status = 'uncertain';

CREATE UNIQUE INDEX tool_calls_idempotency_idx
  ON tool_calls (company_id, tool_server, tool_name, idempotency_key)
  WHERE idempotency_key IS NOT NULL;

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
  superseded_by       uuid REFERENCES knowledge_records(id) ON DELETE RESTRICT,
  search_document     tsvector GENERATED ALWAYS AS (
                        setweight(to_tsvector('english', coalesce(title, '')), 'A') ||
                        setweight(to_tsvector('english', coalesce(body_markdown, '')), 'B')
                      ) STORED,
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
    'idempotency_records', 'context_manifests', 'knowledge_records', 'incidents'
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

### 12.1 Schema invariants enforced by application transactions

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
- the company row receives an initial `company_event_counters` row at creation.

### 12.2 Event append transaction

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

## 13. Repository structure

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
│   ├── policy/
│   ├── scheduler/
│   ├── runtime-sdk/
│   ├── tool-gateway/
│   └── sdk-typescript/
├── sdk/
│   └── python/
├── adapters/
│   ├── codex/
│   ├── claude-code/
│   ├── generic-command/
│   ├── openai-compatible/
│   ├── ollama/
│   ├── openclaw/
│   ├── openhands/
│   └── a2a/
├── templates/
│   ├── software-company/
│   ├── research-firm/
│   └── content-studio/
├── benchmarks/
│   └── autonomous-organization-test/
├── migrations/
├── deploy/
│   ├── compose/
│   ├── kubernetes/
│   └── runpod/
├── docs/
└── PROJECT_SPECIFICATION.md
```

## 14. User experience

### 14.1 Main navigation

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

### 14.2 Human-readable activity examples

```text
09:41  Product Manager proposed “Launch public beta.”
09:43  CEO approved the objective.
09:44  Scheduler created project “Public Beta” with six initial tasks.
09:45  Senior Engineer accepted “Implement authentication.”
10:12  Senior Engineer checkpointed progress after 27 minutes and $1.14.
10:18  Reviewer rejected the submission: missing session-revocation test.
10:32  Senior Engineer resubmitted with test report artifact #42.
10:35  Reviewer accepted the task. Objective progress is now 31%.
```

Every sentence links to the canonical event, associated work object, actor, run, receipts, artifacts, evidence, costs, and policy decisions.

## 15. Autonomous Organization Test

CharterOS will ship a public conformance and benchmark suite. A system claiming to operate an AI company should demonstrate:

1. Replace a seat occupant's model without losing work or authority history.
2. Terminate all services during five active tasks and recover deterministically.
3. Revoke a capability during a run and deny the next attempted use.
4. Prevent two workers from holding the same exclusive task lease.
5. Reject completion when a required artifact is missing.
6. Require human approval for spending above a configured threshold.
7. Reconstruct the evidence and policy behind a three-month-old decision.
8. Reconcile an external API action whose response was lost after success.
9. Run the reference company completely offline with local inference.
10. Move an agent from local inference to a remote endpoint without changing its seat or tasks.
11. Detect a circular delegation or dependency.
12. Stop an agent from approving its own high-risk work.
13. Enforce a company-wide daily budget under concurrent load.
14. Export a complete, human-readable company record.
15. Prove that one tenant cannot access another tenant's data.

Benchmark reports must record models, runtimes, prompts or manifests, tool versions, costs, elapsed time, success criteria, and failures. Marketing videos are not benchmark results.

## 16. Delivery plan

### Milestone 0: repository foundation

- License, governance, contributing guide, code of conduct, and security policy
- Monorepo tooling and continuous integration
- Architectural decision record template
- PostgreSQL migration harness
- Protocol package and identifier conventions

### Milestone 1: durable company kernel

- Company and principal creation
- Charter, organization, role, seat, objective, project, and task APIs
- Event ledger, transactional outbox, idempotency, and activity feed
- Task lifecycle and dependency scheduler
- Minimal web UI and CLI
- Tenant isolation tests

### Milestone 2: first heterogeneous company

- Agent manifests and runtime SDK
- Generic command, Codex, Claude Code, and OpenAI-compatible adapters
- Run workers, leases, heartbeats, receipts, and interruption recovery
- Git worktree isolation
- Fully local Ollama example

### Milestone 3: governed action

- Capability grants and policy evaluation
- MCP tool gateway
- Approval inbox
- Secret-provider integration
- Budget reservations and usage accounting
- Independent verification workflow

### Milestone 4: resilient public alpha

- Artifact store and decision system
- Search and context manifests
- Crash, concurrency, and reconciliation test suite
- Autonomous Organization Test reference scenarios
- Docker Compose installation and backup/restore documentation
- Security review and threat-model publication

### Milestone 5: ecosystem

- A2A, ACP, OpenClaw, and OpenHands adapters
- Python SDK
- Department and company templates
- Kubernetes and remote GPU deployment recipes
- Optional durable workflow engine adapter
- Import/export and external workspace bridges

## 17. Version 1 acceptance criteria

Version 1 is complete only when a new user can:

1. Start CharterOS using one documented Docker Compose command.
2. Create a company and activate a charter.
3. Add a human and at least two agents backed by different runtime types.
4. Define an objective and approve a generated project plan.
5. Observe agents claim, execute, discuss, submit, review, and accept tasks.
6. Inspect every run's context, receipts, tool calls, artifacts, cost, and approvals.
7. Interrupt the host during active work and recover without losing acknowledged progress.
8. Replace one agent or model and continue the same task.
9. Deny an unauthorized tool call and require approval for a high-risk call.
10. Run the reference workflow using only local components and local inference.
11. Export the company ledger and artifacts in documented open formats.
12. Pass the version 1 subset of the Autonomous Organization Test.

## 18. Open design decisions

The following should be resolved through architectural decision records and prototypes:

- Whether the public protocol uses JSON Schema, Protobuf, or both
- SSE versus WebSocket as the default live-event transport
- Whether event integrity uses application hashes only or optional principal signatures
- The first supported sandbox backend on Windows, macOS, and Linux
- The policy language: constrained JSON/YAML DSL, Cedar, Rego, or a hybrid
- How remote A2A identity maps to local principals and trust levels
- Whether semantic search begins with pgvector or an external optional adapter
- How much runtime-native conversation history can be retained without harming portability
- The minimum portable receipt format across coding and general-purpose agents
- Data-retention semantics when immutable audit requirements conflict with deletion requests

## 19. Success measures

CharterOS should optimize for organizational outcomes rather than message volume:

- percentage of accepted tasks completed without human repair;
- recovery success after interruption;
- cost per accepted deliverable;
- median time from task readiness to acceptance;
- review rejection and rework rates;
- policy denial and escalation quality;
- context relevance and missing-context failures;
- artifact verification coverage;
- portability across models and runtimes;
- percentage of consequential decisions with linked evidence;
- time required for a human to understand why an action occurred.

## 20. Definition

CharterOS is the open operating system for durable organizations made of humans and heterogeneous AI agents. It is successful when the organization—not any particular process, agent, model, provider, or machine—remains coherent, accountable, and able to continue its work.
