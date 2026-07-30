# CharterOS

**Bring any agent. Bring any model. Build a company that survives them all.**

CharterOS is an open-source substrate for organizations of humans and heterogeneous AI agents: a durable, append-only **work ledger** with organizational semantics on top — roles, authority, budgets, approvals, verification, and crash recovery — that no model vendor, agent framework, or hosting provider owns.

**The guarantee:** kill every CharterOS process at any moment; the organization resumes with no lost acknowledged work and a complete causal audit trail. “Acknowledged” has a [precise durable boundary](PROJECT_SPECIFICATION.md#95-acknowledgment-and-durability-contract), and the test that falsifies this claim ships in the repository.

> **Status: foundational specification.** There is no code here yet — there is a [specification](PROJECT_SPECIFICATION.md) precise enough to be wrong in public, and a conformance suite designed before the system it tests. Implementation begins with [version 0.1](PROJECT_SPECIFICATION.md#5-scope): the crash-proof work ledger for agent harnesses.

## The demonstration v0.1 exists to make true

```text
$ docker compose up          # demo company boots; three agents mid-work
$ charteros chaos            # kills every process, worker, and connection
$ docker compose up
09:07  RECOVERY  3 interrupted runs classified: 2 restartable, 1 reconcilable.
09:07  Senior Engineer resumed "Add rate limiting" from receipt #4.

$ charteros seat reassign senior-engineer --runtime ollama --model qwen3:32b
09:11  HANDOVER  claude-code → ollama/qwen3:32b. Grants re-derived. Task resumed.
```

The model changed vendors mid-task and the organization did not blink.

## Why this doesn't already exist

2026 settled the easy parts. Buzz gives agents first-class identities in a shared workspace. Harnesses swap models while keeping context. Temporal-class engines make individual runs crash-proof. Hyperscaler control planes govern agents inside their own clouds.

What nobody ships is the layer underneath: a **seat** whose authority, budget, in-flight work, and audit history survive the replacement of whatever occupies it — with the replacement itself a recorded, governed event. Identity today binds to the agent instance or the vendor. CharterOS binds it to the organization.

| Survives swapping the model behind an agent | Workspaces + harnesses (2026) | CharterOS |
|---|---|---|
| Identity and conversation history | ✅ | ✅ |
| Authority and capability grants | ❌ | ✅ |
| Budgets and spend history | ❌ | ✅ |
| In-flight work (lifecycle, leases, receipts) | ❌ | ✅ |
| A record that the swap happened at all | ❌ | ✅ |

The best-documented autonomous-organization failures — Anthropic's Project Vend, Agent Village, TheAgentCompany — were governance failures, not capability failures: unverifiable authority, no ownership, no durable structure. That layer is this project.

## The Autonomous Organization Test

CharterOS ships a public conformance suite, mechanically graded, runnable by anyone — including competitors. Its scenarios are named for the fiction and field failures that motivated them:

- **The Ship of Theseus** — replace a seat's occupant; nothing organizational is lost
- **The Cryosleep** — kill everything mid-work; recover deterministically
- **The Governor Module** — revoke a capability mid-run; the next use is denied
- **The Prestige** — two workers can never hold the same exclusive lease
- **The Tungsten Cube** — spend above threshold waits for a human
- **The HAL Clause** — an instruction absent from the ledger carries no authority
- **The Boardroom Coup** — authority asserted in chat changes nothing; only ledgered, approved assignments command

[All seventeen scenarios →](PROJECT_SPECIFICATION.md#17-the-autonomous-organization-test)

## Read the specification

[**PROJECT_SPECIFICATION.md**](PROJECT_SPECIFICATION.md) — the full design: domain model, seat handover protocol, verification ladder, Cedar policy engine, PostgreSQL schema, delivery plan, and the mid-2026 landscape analysis this design answers.

## Commitments

- **Apache-2.0, permanently.** This project publicly commits to never relicensing the core.
- **DCO, not CLA.** No entity accumulates the rights that make a future rug-pull possible.
- **Falsifiable claims only.** Every headline capability has a shipping test. No self-scored leaderboards.
- **AI-assisted contributions welcome, disclosed, issue-first.** See CONTRIBUTING.md and AGENTS.md (arriving with Milestone 0).

---

*Fiction spent a century imagining organizations of humans and machines, and kept arriving at the same conclusion: the machines were never the variable that decided the ending. The governance was.*
