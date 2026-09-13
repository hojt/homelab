# ADR-0008: Platform Development Workspace

**Status:** Accepted

**Refines:** ADR-0001

## Context

The homelab separates architecture, platform lifecycle, desired environment
state, and application source code into repositories with explicit
responsibilities.

This separation keeps ownership boundaries clear, but platform development
frequently requires reasoning across several repositories at the same time.

For example, a platform change may require:

- architecture context from `homelab`
- implementation changes in `local-platform`
- desired-state changes in `local-environments`

Working with only one of these repositories at a time makes cross-repository
changes harder to understand and validate as a whole.

At the same time, combining the repositories into a monorepo would weaken the
responsibility boundaries established by ADR-0001.

The project also uses coding agents. A coding agent benefits from access to the
same cross-repository context as the human developer, but should not
automatically receive the privileged host and cluster capabilities required for
platform operation.

A development workspace therefore needs to be broader than an individual Git
repository without turning the participating repositories into a single
ownership or versioning unit.

## Decision

A dedicated `platform-workspace` repository provides the development workspace
for the homelab platform.

The workspace composes these independent repositories:

| Repository | Responsibility |
|------------|----------------|
| `homelab` | Own architectural documentation and decisions. |
| `local-platform` | Bootstrap and operate the local Kubernetes platform. |
| `local-environments` | Own GitOps desired state for Kubernetes environments. |

The repositories remain normal, independently versioned Git repositories with
their own history, branches, staging areas, commits, tags, and remotes.

The `platform-workspace` repository owns only workspace-level concerns,
including:

- workspace documentation and agent instructions
- developer and coding-agent Dev Container configuration
- workspace entrypoints such as `dev.sh` and `agent.sh`
- automation that genuinely spans the workspace, such as repository status and
  diff inspection

Repository-specific implementation, documentation, and automation remain in
their owning repositories.

The repositories are kept as siblings on the host and bind-mounted into the
workspace under `repos/`.

Git submodules or other mechanisms that make the workspace repository track
specific commits of the participating repositories are not used. Such coupling
is deferred until a concrete need for coordinated repository version pinning is
demonstrated.

The workspace does not include application repositories such as `example-app`.
Application development uses separate application workspaces as described by
ADR-0007.

The developer and coding-agent environments expose the same repository context
but intentionally have different capabilities.

The developer environment may receive the host capabilities required to operate
the local platform, including access to the container runtime and Kubernetes
configuration.

The coding-agent environment receives read/write access to the workspace
repositories but does not receive host container-runtime sockets, Kubernetes
credentials, host Git configuration, host networking, or similar privileged
capability channels by default.

Platform command-line tools may still be present in the coding-agent image so
that repository content can be inspected and statically validated without
granting control over the host or live cluster.

The workspace entrypoint verifies the expected agent repository mounts and
containment properties before starting the coding agent.

## Consequences

Platform architecture, implementation, and desired state can be reasoned about
and changed from one development workspace.

Cross-repository platform changes become easier to review because all relevant
repositories are visible together.

Repository responsibility boundaries remain explicit because the repositories
retain independent Git state and continue to own their respective content.

The workspace repository adds a small amount of coordination configuration and
assumes a known sibling-repository layout on the host.

Workspace-level automation may span the participating repositories, while
repository-specific automation remains local to each repository.

The developer environment can operate the local platform without requiring the
coding-agent environment to have the same level of host or cluster access.

The coding agent can inspect architecture, edit implementation and desired
state, and perform static validation while its blast radius remains limited.

The workspace does not provide atomic commits across repositories. A change
that spans repositories is still reviewed and committed separately in each
repository by the human developer.

The project explicitly distinguishes between a Git repository, a development
workspace, and a deployment unit. These boundaries may align in some cases but
are not treated as equivalent.
