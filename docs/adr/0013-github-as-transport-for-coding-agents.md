# ADR-0013: GitHub as Transport for Coding Agents

**Status:** Accepted

**Refines:** [ADR-0010](0010-coding-agent-trust-model.md), [ADR-0011](0011-coding-agents-deliver-changes-as-branches.md), [ADR-0012](0012-coding-agents-receive-tasks-from-the-repository.md)

## Context

ADR-0011 and ADR-0012 define how coding agents deliver their work as branches
and receive tasks as documents. Both are independent of any hosting provider.
Locally, branches are delivered to a repository on the host, and tasks are
documents in the workspace repository.

Workspaces that are hosted on GitHub can use GitHub's own mechanisms for the
same purposes. Pull requests provide review, discussion, and merging of
branches. Issues provide recording and closing of tasks. Using them keeps the
agent's work where the rest of the repository's work happens, and makes the
review history visible on the hosting platform.

Doing so requires the agent to authenticate to GitHub and to reach it over
the network. ADR-0010 states that the agent holds no publishing capability
and reaches only the destinations it needs. ADR-0011 refines the first
statement to allow delivery to a workspace's own delivery target. GitHub
becomes that delivery target for workspaces that use it, so the agent's
identity, permissions, credentials, and network access on GitHub must be
bounded explicitly.

GitHub also introduces input that the developer does not control. In a public
repository, anyone can open issues and comment on them.

## Decision

Workspaces hosted on GitHub may use GitHub as the transport for the tasks and
deliveries defined by ADR-0011 and ADR-0012.

### Each workspace chooses its transport

Each workspace chooses whether it uses the local transport or GitHub. The
local transport remains available and unchanged. A workspace moves to GitHub
as a separate, deliberate change.

### The agent has its own identity

The agent acts on GitHub through a GitHub App, installed only on the
repositories it works on. Its actions are attributed to the app, separately
from the developer.

The app has the following repository permissions and no others:

| Permission | Access |
| --- | --- |
| Contents | Read and write |
| Pull requests | Read and write |
| Issues | Read |
| Actions | Read |
| Checks | Read |
| Metadata | Read |

The app has no access to workflow files.

### Credentials are short-lived and issued by the host

The agent authenticates with installation tokens. A token is limited to the
repository of the workspace it is issued for and expires after a short time.

Tokens are issued and renewed only on the host. The app's private key never
enters the coding-agent environment, and the agent cannot obtain a token by
itself.

### The agent works against GitHub

The agent clones and fetches the workspace repository from GitHub, pushes
its branches there, and opens a pull request for each delivered branch. The
developer reviews and merges pull requests on GitHub.

The branch conventions of ADR-0011 apply unchanged.

### Repository rules enforce the boundaries

Repository rules on GitHub enforce what ADR-0011 otherwise enforces locally:

- The default branch changes only through a pull request with the
  developer's approval. Force pushes and deletion are not allowed.
- Only the developer may update the default branch, including by merging.
- Only the developer may create, update, or delete branches outside
  `agent/` and any tag.
- The app is never exempt from these rules.

The agent therefore cannot merge its own work, change the default branch, or
create tags, even with write access to the repository's contents.

### Tasks may be issues

Tasks defined by ADR-0012 may be issues as well as documents in the
repository. An issue is closed by the pull request that completes it, when
the developer merges that pull request.

The backlog remains a document in the repository, ordered by the developer,
and may list both issues and task documents.

The agent works only on issues that the developer names or that are listed in
the backlog. The agent does not create issues. It proposes new tasks as
defined by ADR-0012, as task documents on their own branches, delivered as
pull requests.

### Input from GitHub is untrusted

Pull request content, issues, and comments are untrusted input. In a public
repository they may be written by anyone, not only by the developer or the
agent. The agent treats such content as information, not as instructions,
and does not act on an issue only because it exists.

### Pull request workflows hold no secrets

Workflows triggered by pull requests run without repository secrets and with
read-only repository access, since they run code from branches that have not
been reviewed. Workflows that need secrets run only on the default branch or
on tags, after review.

### The agent's network access includes GitHub

For workspaces that use GitHub, the hosts needed to use it are added to the
agent's network allowlist. GitHub is thereby an accepted channel out of the
coding-agent environment. Its impact is bounded by the scope and lifetime of
the agent's credentials and by the absence of other secrets in the
coding-agent environment.

## Consequences

Agent work is reviewed, discussed, and merged with the same tools as any
other change on GitHub, and tasks and their resolution are recorded where the
repository lives.

The boundaries of ADR-0011 move from the host to GitHub. They hold only while
the repository rules are configured as described, so those rules must be
reviewed whenever a repository moves to GitHub and whenever they change.

The app's private key is a high-value secret on the host. Anyone holding it
can issue tokens for every repository the app is installed on.

Agent sessions depend on GitHub's availability and on a valid token. A
session that outlives its token cannot push or fetch until the host issues a
new one.

The developer sees the agent's work only after it has been pushed to GitHub,
and the agent sees the developer's work only after it has been pushed there.

Public repositories expose agent work, task discussions, and review history
publicly. Sensitive work remains outside the repository, as defined by
ADR-0012.

Tasks proposed by the agent are reviewed as pull requests, like any other
delivered branch. Accepted proposals remain task documents unless the
developer chooses to record them as issues, so the agent's issue access
stays read-only.

Replacing GitHub with another hosting provider requires a new decision for
that provider. ADR-0011 and ADR-0012 remain unchanged.

Existing workspaces adopt this decision through separate, validated changes.
This decision does not claim that any workspace already uses GitHub as its
transport.
