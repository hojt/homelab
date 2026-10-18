# ADR-0011: Coding Agents Deliver Changes as Branches

**Status:** Accepted

**Refines:** [ADR-0008](0008-platform-development-workspace.md), [ADR-0009](0009-application-workspace-composition.md), [ADR-0010](0010-coding-agent-trust-model.md)

## Context

ADR-0010 contains the coding agent in a sandbox. In application workspaces,
the developer's working tree is mounted into the coding-agent environment.
The workspace is read-only, with selected paths writable, so the agent writes
its changes directly into the developer's working tree. Git metadata is
read-only, and ADR-0008 and ADR-0009 establish that the human developer owns
staging, commits, tags, and pushes.

This arrangement has several drawbacks:

- The developer's working tree is shared with an untrusted environment.
  Containment depends on a growing set of mounts that protect files executed
  or interpreted elsewhere, and on how those mounts behave when files are
  replaced on the host.
- Agent changes and the developer's own changes are mixed in one working
  tree. Reviewing what the agent did means separating it from everything
  else that is uncommitted.
- The agent cannot record its work as commits, so its changes have no
  history, no description, and no unit of review smaller than the whole
  working tree.

Pull requests address the same problems on hosting platforms: work happens on
a separate branch, and it reaches the main line only through review. The same
model can be applied locally, independently of any hosting provider.

## Decision

Coding agents work in their own clone and deliver their changes as branches.

### The agent works in its own clone

The developer's working tree is not mounted into the coding-agent
environment. The agent works in a clone of the workspace repository, kept in
storage that belongs to the coding-agent environment of that workspace.

The clone is created from the workspace repository, which is made available
to the agent read-only as a source to clone and fetch from. The agent sees
committed history only. Uncommitted work in the developer's working tree is
not visible to the agent.

### The agent commits on its own branches

The agent may create commits, but only in its own clone and only on branches
with a reserved prefix, `agent/`. Agent commits are authored with an identity
that distinguishes them from the developer's commits.

The human developer remains the only actor that changes the developer's
repositories, their main and integration branches, their tags, and their
remotes.

### Branches reach the developer through a delivery target

The agent publishes its branches to a delivery target dedicated to the
workspace. The developer fetches branches from the delivery target, reviews
them, and decides whether to merge them.

Nothing produced by the agent reaches the developer's repositories without an
explicit fetch, review, and merge by the human developer.

Initially, the delivery target is a local repository that only the
coding-agent environment of the workspace can write to. A hosting provider
may replace it later. Such a change affects only the delivery target; the
rest of this decision is independent of where branches are delivered.

### Delivered branches are untrusted input

Branches fetched from the delivery target are agent output and are untrusted
until reviewed, as defined by ADR-0010. Fetching does not execute their
content. Building, testing, or otherwise executing a delivered branch in the
developer environment happens only after review.

## Consequences

The developer's working tree, Git metadata, and host-executed files are no
longer reachable from the coding-agent environment. Protecting them no longer
depends on read-only mounts inside a shared working tree.

Each piece of agent work becomes a branch with its own history and commit
messages, which can be reviewed, compared, merged, or discarded on its own.

ADR-0008 and ADR-0009 state that the human developer owns commits. This
decision refines that: the developer owns all commits in the developer's
repositories, while the agent may commit only in its own clone, on its own
branches. ADR-0010 states that the agent holds no publishing capability.
This decision refines that as well: the agent may publish only to the
delivery target of its own workspace, never to the developer's repositories
or their remotes.

The agent no longer sees uncommitted work. Sharing context with the agent
requires committing it first, or describing it in the task.

Reviewing agent work requires fetching and merging branches, an additional
step compared with reviewing changes in the working tree. Workspace tooling
should make this step simple.

The delivery target is writable by the coding-agent environment and read by
the developer. It must not be a location from which anything is executed on
the host.

Agent state, including the clone and the delivery target, belongs to the
workspace. Removing that state discards undelivered agent work but does not
affect the developer's repositories.

Existing workspaces adopt this decision through separate, validated changes,
starting with application workspaces. This decision does not claim that any
workspace already follows it.
