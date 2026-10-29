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
supersedes or refines.

Accepted ADRs are not rewritten. Facts discovered while implementing a
decision may be added in a dated entry under an Addenda section at the
end of the ADR. Changing the decision itself requires a new ADR that
refines or supersedes it.

See [`TEMPLATE.md`](./TEMPLATE.md) when creating a new ADR.

## Records

ADR-0010 to ADR-0014 together define how coding agents are contained, deliver
their work, receive tasks, use GitHub, and share infrastructure through the
template. See [`../coding-agents.md`](../coding-agents.md) for an overview of
how they fit together, and for the implementation choices, limitations, and
next steps that the ADRs do not record.

| ADR | Title |
|-----|-------|
| [ADR-0001](./0001-repository-structure.md) | Repository Structure |
| [ADR-0002](./0002-development-environment-philosophy.md) | Development Environment Philosophy |
| [ADR-0003](./0003-repository-self-containment.md) | Repository Self-Containment |
| [ADR-0004](./0004-incremental-development.md) | Incremental Development |
| [ADR-0005](./0005-gitops-architecture.md) | GitOps Architecture |
| [ADR-0006](./0006-persist-secret-encryption-key-material.md) | Persist secret encryption key material |
| [ADR-0007](./0007-application-workspace-structure.md) | Application Workspace Structure |
| [ADR-0008](./0008-platform-development-workspace.md) | Platform Development Workspace |
| [ADR-0009](./0009-application-workspace-composition.md) | Application Workspace Composition |
| [ADR-0010](./0010-coding-agent-trust-model.md) | Coding Agent Trust Model |
| [ADR-0011](./0011-coding-agents-deliver-changes-as-branches.md) | Coding Agents Deliver Changes as Branches |
| [ADR-0012](./0012-coding-agents-receive-tasks-from-the-repository.md) | Coding Agents Receive Tasks From the Repository |
| [ADR-0013](./0013-github-as-transport-for-coding-agents.md) | GitHub as Transport for Coding Agents |
| [ADR-0014](./0014-application-workspaces-follow-the-template.md) | Application workspaces follow the template |

