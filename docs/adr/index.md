# Architecture Decision Records

Architecture Decision Records (ADRs) capture important architectural decisions
made during the evolution of the homelab.

An ADR documents **why** a decision was made rather than **how** it is
implemented.

Each ADR describes:

- the context
- the decision
- the consequences

## Philosophy

Keep ADRs short and focused.

Record architectural decisions rather than implementation details.

An ADR should typically describe a decision that:

- affects multiple repositories,
- is expected to have a long lifetime,
- would be difficult or expensive to change later, or
- is likely to be questioned or revisited in the future.

Implementation details, tooling choices, and day-to-day development decisions
generally do not require an ADR.

## Lifecycle

ADRs are written after a decision has been made and accepted.

Prefer creating a new ADR over modifying an existing one.

If an architectural decision changes, create a new ADR and reference the one it
supersedes.

See `TEMPLATE.md` when creating a new ADR.
