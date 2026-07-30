# ADR-0002: Policy language is Cedar behind a constrained YAML surface

**Status:** Accepted · 2026-07-30

## Decision

CharterOS evaluates authorization policy with **Cedar**. Policies are authored in the constrained YAML surface shown in the specification (§11.2) and compiled to Cedar at activation. Raw Cedar is an operator escape hatch, not the authoring default.

## Context

Policy semantics leak into capability grants, permission envelopes, seat authority, and the approval engine — deferring the choice would calcify a dozen tables around an accidental engine. Cedar's principal–action–resource–condition model maps directly onto CharterOS grants; it is deterministic, total, side-effect free, and statically analyzable. Rego is a general-purpose language without those guarantees by construction; a bespoke DSL would accrete escape hatches until it became a bad programming language.

## Consequences

- Policy simulation ("what would this change have blocked?") gets Cedar's analysis tooling for free.
- The YAML surface must stay a strict subset — anything inexpressible in it requires a deliberate escape to raw Cedar, which is itself a policied act.
- Cedar entity modeling becomes part of the protocol package.
