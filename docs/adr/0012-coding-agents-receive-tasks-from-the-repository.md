# ADR-0012: Coding Agents Receive Tasks from the Repository

**Status:** Accepted

**Refines:** [ADR-0011](0011-coding-agents-deliver-changes-as-branches.md)

## Context

ADR-0011 defines how coding agents deliver their work: as branches, fetched,
reviewed, and merged by the human developer. It does not define how work
reaches the agent. Today, tasks are given in the conversation that starts an
agent session. They are not recorded anywhere, cannot be planned or ordered
ahead of time, and leave no trace connecting a delivered branch to the need
it addresses.

Hosting platforms solve this with issue trackers: tasks are recorded,
discussed, prioritized, and closed when the change that resolves them is
merged. The same model can be applied locally, independently of any hosting
provider, in the same way that ADR-0011 applies pull requests locally.

The agent already sees everything the developer has committed, and the
developer already reviews everything the agent delivers. A task mechanism
that uses those two paths needs no additional channel between the two
environments.

## Decision

Coding agents receive tasks as documents in the workspace repository, and
close them through the same branches that deliver the work.

### Tasks are documents in the repository

Each task is a document in a dedicated location in the workspace
repository, identified by a short descriptive name. A task describes its
goal and its acceptance criteria, and records its status.

A task has one of the following statuses:

| Status | Meaning |
| --- | --- |
| `proposed` | Suggested by the agent and not yet accepted by the developer. |
| `open` | Accepted and ready to be worked on. |
| `done` | Completed by a merged change. |
| `closed` | Not going to be done. |

The agent sees tasks through the committed history it already receives, as
defined by ADR-0011. A task is available to the agent once the developer has
committed it.

### The developer orders the work

An ordered backlog in the same location lists the tasks that should be done,
in the order they should be done. The developer owns the backlog.

The agent works on a task the developer names. When asked to take the next
task, it takes the first open task in the backlog. Open tasks that are not
in the backlog are worked on only when the developer names them.

### A task is closed by the change that completes it

The agent works on one task per branch, named after the task, following the
branch conventions of ADR-0011.

In the same branch, the agent sets the task's status to `done` and removes
it from the backlog. The task is closed when the developer merges the
branch, so the task, the change, and its review are recorded together.

### The agent may propose tasks

The agent may propose new tasks, for example when it discovers work while
completing another task. Proposals are delivered on their own branches, with
the status `proposed`, and do not change the backlog.

The developer accepts a proposal by setting its status to `open`, placing it
in the backlog if appropriate, and merging it. The developer rejects a
proposal by not merging it.

### Proposed tasks are untrusted input

A task is an instruction to future agent sessions. A proposed task is agent
output and is untrusted until reviewed, as defined by ADR-0010. Reviewing a
proposal includes reviewing its text as instructions, not only as a
description of work.

### Sensitive tasks are kept out of the repository

Workspace repositories may be public. Tasks that cannot be described without
disclosing sensitive information, such as unresolved security issues, are
tracked outside the repository. A task in the repository may refer to such
work only in terms that disclose nothing sensitive.

### The mechanism is independent of the hosting provider

This decision defines how tasks are recorded, ordered, and closed, not where
they are stored. A hosting provider's issue tracker may later replace the
documents in the repository. Such a change affects only where tasks are
stored; the lifecycle, ownership, and review defined here remain.

## Consequences

Tasks are recorded, versioned, and reviewable, and each delivered branch is
connected to the task it completes.

No new channel between the developer environment and the coding-agent
environment is needed. Tasks reach the agent through committed history, and
task updates reach the developer through delivered branches.

The agent cannot change the status of a task in the developer's repository.
A task in progress is visible only as a delivered branch named after it, and
its status changes only when the developer merges that branch.

Tasks and their history are public whenever the repository is public.

Branches that complete different tasks may each change the backlog. When one
is merged, the other may need to be rebased, which the agent resolves as part
of updating its branch.

Each proposed task is an additional branch to review. Review is the only
point at which agent-proposed instructions are accepted.

The developer ranks work only through the order of the backlog. Tasks carry
no priority of their own.

Existing workspaces adopt this decision through separate, validated changes,
starting with application workspaces. This decision does not claim that any
workspace already follows it.
