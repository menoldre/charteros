# AGENTS.md

Guidance for AI agents working in this repository — including, eventually, CharterOS's own demo company, which triages this repository's issues.

## Current state

This repository is at the **specification stage**. There is no build, no test suite, and no code. The authoritative artifacts are:

- `PROJECT_SPECIFICATION.md` — the full design. Read the relevant section before proposing changes; section numbers are stable within a revision.
- `adr/` — accepted architecture decisions. Do not propose changes that contradict an accepted ADR without proposing a superseding ADR.
- `README.md` — positioning and public commitments. The guarantee and the license commitments are load-bearing; never weaken them in an edit.

## Rules for agent contributions

1. **Issue-first, PR-second.** Open or reference an acknowledged issue before submitting changes.
2. **Disclose AI assistance** in the PR description. Undisclosed AI-generated contributions are closed without review.
3. **Sign off with DCO** (`git commit -s`). This project has no CLA and never will.
4. **Do not fabricate.** Every factual claim added to the specification (market, protocol versions, citations) must carry a verifiable source. Claims that cannot be verified are marked as uncertain or omitted.
5. **Consistency over cleverness.** Match the specification's voice: declarative, terse, falsifiable. If a sentence cannot be wrong, it does not belong in the spec.
6. **Schema changes** must preserve the invariants of spec §14.1 — tenant-composite foreign keys, append-only tables, RLS coverage for every new table.

## When code exists (Milestone 0+)

This file will be extended with build, test, and conformance-run instructions. Until then, treat the specification as the codebase.
