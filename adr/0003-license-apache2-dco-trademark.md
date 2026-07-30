# ADR-0003: Apache-2.0 forever, DCO not CLA, trademark as the moat

**Status:** Accepted · 2026-07-30

## Decision

- License: **Apache-2.0, with a public commitment to never relicense the core.**
- Contributions: **DCO** sign-off. No CLA, ever.
- The **trademark** is registered and a trademark policy published; conformance and the name stay with the community process even if the code is forked.

## Context

Post-2023 relicensing events (HashiCorp, Redis, Elastic) taught buyers that the question is not "which license" but "who can change it." A CLA is precisely the aggregation mechanism that enables a future rug-pull; refusing one is the strongest trust signal a single-vendor project can send. The durable moat for infrastructure is the mark and the conformance program, in the Kubernetes tradition — not the code.

## Consequences

- Monetization, if pursued, is managed hosting and organization-tier compliance packaging — never feature-gating what developers touch, and never the audit trail, which is the core free product.
- If neutrality becomes the constraint, the protocol and conformance suite are what gets donated to a foundation (the Agentic AI Foundation is the natural home), not necessarily the product.
