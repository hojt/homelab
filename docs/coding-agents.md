# Coding Agents in the Homelab

This document describes how coding agents are used in the homelab: what the
work aims for, how the pieces fit together, which decisions have been made and
why, what was considered and rejected, and what remains open.

It is a living overview. Decisions themselves are recorded in ADRs, which are
not rewritten; this document summarizes them, links to them, and records the
implementation choices, known limitations, and next steps that ADRs do not
cover. Update it when the design changes.

## Purpose and goals

The homelab is a learning environment for development and DevOps. Coding
agents are part of that learning: the aim is to use them for real work in
application and platform workspaces, while understanding and controlling what
they can do.

The goals are:

- **Contain the agent.** An agent acts on untrusted input and may be
  manipulated or simply wrong. Its reach must be bounded by the environment,
  not by its good behavior.
- **Keep a human at every boundary.** Nothing the agent produces reaches the
  developer's repositories, the host, or the platform without review.
- **Make agents useful.** Containment should not prevent the agent from
  building, testing, and delivering changes, or from receiving tasks.
- **Keep it maintainable.** Workspaces share one implementation, and
  improvements reach every workspace without manual copying.
- **Stay provider-independent.** The workflow works locally, and a hosting
  provider such as GitHub is an optional transport.

## Scope

In scope:

- The coding-agent sandbox in the application workspaces (`example-app`,
  `template-app-workspace`, and applications created from the template) and in
  `platform-workspace`.
- How agents receive tasks and deliver work, locally and through GitHub.
- How workspace infrastructure is shared and updated through the template.

Out of scope:

- The platform itself (`local-platform`, `local-environments`), except where
  the sandbox protects platform secrets.
- Agent access to the Kubernetes cluster. It is an open question, not part of
  the current design.
- Production use. The design targets a single developer's homelab.

## Overview

### Trust zones

ADR-0010 defines the trust model:

| Zone | Trust |
| --- | --- |
| Host | Trusted. Runs host entrypoints and host scripts. |
| Developer environment | Trusted. Human-driven Dev Container with platform access. |
| Platform | Trusted. Kubernetes, its registry, and GitOps environments. |
| Coding-agent environment | Untrusted. Runs the agents and their commands. |
| External services | External. Model providers and package registries. |

Agent output is untrusted until the developer has reviewed it. Content reaches
a more trusted zone only through a deliberate human action.

### Workspaces and entrypoints

Each workspace has two host entrypoints:

- `dev.sh` starts the developer environment.
- `agent.sh` starts the coding-agent environment, verifies its containment,
  and starts an agent tool: OpenCode by default, or Claude Code with
  `./agent.sh claude`.

`agent.sh` is identical in all workspaces. What differs is described in
`.devcontainer/agent/sandbox.conf`.

### How an agent works in an application workspace

1. The developer records a task as a document in `tasks/`, or as a GitHub
   issue, orders it in `tasks/BACKLOG.md`, and commits it.
2. The agent works in **its own clone** of the repository, never in the
   developer's working tree. It sees committed history only.
3. The agent works on a branch named `agent/<task-name>`, and closes the task
   in the same branch.
4. The agent delivers the branch: to a local delivery repository, fetched with
   `./agent.sh fetch`, or to GitHub as a pull request.
5. The developer reviews, merges, and publishes. Nothing is merged
   automatically.

### Containment layers

The coding-agent environment combines several independent layers, each
verified by `agent.sh` before an agent starts:

- **Filesystem.** In application workspaces, the agent works in its own clone
  in an agent-only volume; the developer's working tree, Git metadata, and
  host scripts are not mounted. In `platform-workspace`, repositories are
  mounted read-only, with selected paths writable and local key material
  hidden.
- **Agent-only state.** Caches, build output, agent logins, and agent tool
  state are in per-workspace volumes or tmpfs mounts, never shared with the
  developer environment or other workspaces.
- **Privileges.** Rootless Podman, all capabilities dropped,
  `no-new-privileges`, memory, CPU, and process limits, and no Podman in the
  agent image.
- **Network.** The agent container has no route out. All traffic goes through
  an egress proxy that allows only listed hosts and logs every request.
- **Credentials.** No host credentials, SSH agent, Kubernetes configuration,
  or container runtime access. The only credentials are the agent tools'
  logins and, with GitHub, a short-lived token.
- **Supply chain.** The base image is pinned by digest, downloaded binaries
  are verified against pinned checksums, and agent tools are installed from a
  lockfile.
- **Agent permissions.** OpenCode and Claude Code are configured to ask before
  destructive commands and to never publish. These are conveniences; the
  layers above carry the containment.

## Decisions

The decisions are listed in [`adr/index.md`](adr/index.md).

### [ADR-0010](adr/0010-coding-agent-trust-model.md): Coding agent trust model

The agent is an untrusted actor. Nothing executed in a more trusted zone is
writable by it, it holds no publishing capability beyond what later ADRs
allow, it cannot reach platform or host services, it shares no writable state
with more trusted zones, and it sees only the secrets it needs. Containment is
verified and fails closed.

*Why:* containing the agent by its environment is the only approach that holds
when the agent is manipulated or wrong. Prompts and instructions help, but
cannot be relied on.

### [ADR-0011](adr/0011-coding-agents-deliver-changes-as-branches.md): Agents deliver changes as branches

The agent works in its own clone and delivers branches with the prefix
`agent/` to a delivery target. The developer fetches, reviews, and merges.

*Why:* it removes the developer's working tree, Git metadata, and host scripts
from the agent's reach entirely, instead of protecting them file by file. It
also gives agent work a history and a unit of review. The delivery target is
replaceable, so the same model works locally and with a hosting provider.

### [ADR-0012](adr/0012-coding-agents-receive-tasks-from-the-repository.md): Agents receive tasks from the repository

Tasks are documents in `tasks/`, with the statuses `proposed`, `open`, `done`,
and `closed`. The developer owns the order in `tasks/BACKLOG.md`. A task is
closed by the branch that completes it. The agent may propose tasks on
separate branches, and proposals are untrusted input.

*Why:* tasks reach the agent through the committed history it already sees,
and task updates reach the developer through branches it already reviews. No
new channel between the environments is needed.

### [ADR-0013](adr/0013-github-as-transport-for-coding-agents.md): GitHub as transport for coding agents

Workspaces may choose GitHub instead of the local transport. The agent acts as
a GitHub App with minimal permissions, uses short-lived tokens issued only on
the host, and delivers pull requests. Repository rules keep the default
branch, other branches, and tags under the developer's control. Tasks may also
be issues; the agent reads them but does not create them. Input from GitHub
is untrusted. Pull request workflows hold no secrets.

*Why:* review and task tracking happen where the repository lives, with a
separate, attributable identity for the agent, and without giving it any
long-lived credentials.

Addendum: on repositories owned by a personal account, the developer merges by
explicitly bypassing the rules after approving, since no rule can let only the
developer merge.

### [ADR-0014](adr/0014-application-workspaces-follow-the-template.md): Application workspaces follow the template

Applications keep `template-app-workspace` as an upstream and merge its
tagged versions with Git. The template declares which files it owns;
applications do not change them. Updates are merge commits, never squashed or
rebased. `example-app` is an application created from the template.

*Why:* improvements to the shared infrastructure reach every application,
including those created earlier, and are made in one place. Git merges keep
history and show exactly where an application has diverged.

## Implementation choices

These choices are not ADRs, but explain how the decisions are implemented.

- **One `agent.sh` everywhere.** Workspace differences are in `sandbox.conf`,
  which declares the workspace mode (`clone` or `mount`), the transport
  (`local` or `github`), and, in mount mode, which paths are writable,
  protected, isolated, or hidden.
- **Checks run in one exec.** All containment checks are queued on the host
  and run in a single command in the agent container. The host counts the
  results, so a failed exec fails the verification instead of passing negated
  checks.
- **One agent `devcontainer.json` for both transports.** `agent.sh` sets the
  sources of the mounts through environment variables: the developer's `.git`
  or an empty directory, and the token directory or an empty directory.
  Switching transport requires `./agent.sh rebuild`.
- **Fetching through a bundle.** `./agent.sh fetch` has the agent container
  create a Git bundle of `agent/*` branches; the host only fetches from the
  bundle and never runs Git inside the agent-controlled delivery repository.
  It works from the host and from the developer container.
- **The egress proxy.** tinyproxy on an internal network, with an allowlist
  assembled when the proxy starts: the template's shared list, GitHub hosts
  when the transport is GitHub, and the application's
  `sandbox.allowlist`. `./agent.sh proxy-log` summarizes allowed and refused
  hosts from journald.
- **Agent logins.** OpenCode logs in once per workspace, into a
  per-workspace volume. Claude Code uses one long-lived token from
  `claude setup-token`, stored on the host and passed only to the Claude Code
  process; its other state is per workspace. Claude Code's permissions are in
  managed settings in the image, which hooks or user settings cannot override.
- **Agent instructions.** Shared instructions are in
  `.devcontainer/agent/instructions.md`, owned by the template.
  Application-specific instructions are in `AGENTS.md`. Both tools load both
  files explicitly at startup.
- **GitHub tokens.** The host signs a JWT with the app's private key, using
  `openssl` from the host or a toolbox, and requests an installation token
  limited to the workspace's repository and permissions. The token is written
  to `$XDG_RUNTIME_DIR` and mounted read-only. Git reads it through a
  credential helper and `gh` through a wrapper, so it never enters the
  container's environment. Tokens are renewed manually with
  `./agent.sh github-token`.
- **Template updates.** `task template:update -- <version>` fetches a template
  tag into `refs/template/`, outside the application's tags, merges it, and
  keeps content the application has removed. `task template:status` shows the
  versions an application has used.

## Alternatives considered and rejected

- **Protecting individual files with read-only bind mounts.** Replacing a
  file on the host, as editors do when saving, detaches the mount and exposes
  the writable file underneath. Read-only directories and, later, agent-owned
  clones replaced this approach.
- **Mounting the developer's working tree into the agent environment.** Even
  read-only with writable exceptions, it exposed Git metadata, host scripts,
  and local secrets, and depended on a growing list of mounts. Replaced by
  agent-owned clones (ADR-0011).
- **A firewall inside the agent container.** It requires capabilities such as
  `NET_ADMIN`, which conflicts with dropping all capabilities. An external
  proxy on an internal network needs no privileges and fails closed.
- **Re-enabling SELinux through a Podman wrapper.** The Dev Container CLI
  always disables SELinux labeling for Podman. A wrapper that strips the
  option was rejected as an undocumented workaround; the limitation is
  documented instead (ADR-0010 addendum).
- **Sharing one Claude Code configuration across workspaces.** It made memories
  and settings a writable channel between workspaces. Replaced by a shared
  token and per-workspace state.
- **Using a Claude or ChatGPT subscription through another tool.** Not
  permitted by the providers' terms. Each subscription is used with its own
  tool.
- **A personal access token or a machine account for GitHub.** A personal
  token acts as the developer and lives long; a machine account always gets
  full write access to personal repositories. A GitHub App gives a separate
  identity and short-lived, scoped tokens.
- **GitHub App user tokens.** They act as the developer, which breaks review:
  the developer cannot approve the agent's pull requests, and the agent could
  inherit the developer's rule exemptions.
- **Letting the agent create GitHub issues.** Write access to issues cannot be
  limited to creating them. Agent proposals are task documents on branches
  instead.
- **Priority fields on tasks.** Priorities change over time; an ordered
  backlog owned by the developer is simpler.
- **Merging `example-app` and the template into one repository.** New
  applications would inherit everything added to the example. Keeping the
  template as an upstream solves duplication and also brings updates to every
  application.
- **Synchronizing template files with a script instead of Git merges.** It
  avoids conflicts but loses history and Git's view of divergence. Git merges
  were chosen, with removed content kept removed.
- **Rebasing applications onto the template.** It rewrites published history
  and tags.
- **Moving or reusing a published template tag.** Published tags are treated
  as immutable; a correction is a new version.
- **Running `agent.sh` modes or other tooling from the developer container.**
  Only `fetch` is supported there; everything else runs on the host, where
  `agent.sh` belongs.

## Known limitations and accepted risks

- **SELinux is disabled for the agent container** by the Dev Container CLI.
  Containment relies on the remaining layers.
- **Allowed hosts are channels out.** The model providers, package registries,
  and, with GitHub, GitHub itself can carry data out of the agent environment.
  The impact is bounded by the absence of secrets and by scoped, short-lived
  credentials.
- **Agent-written code runs in the developer environment** when the developer
  builds or tests it. Review before execution is the only protection, and the
  developer environment has platform access.
- **Agent tool logins are readable by the agent.** They are dedicated to agent
  use and revocable.
- **Prompts are not containment.** An agent can change its own OpenCode
  configuration in its clone; such changes appear in delivered branches.
- **GitHub tokens expire after an hour.** Longer sessions need
  `./agent.sh github-token`.
- **Merging on personal GitHub accounts requires bypass.** Approval before
  merging is the developer's discipline, not enforced.
- **Instructions take effect only when the agent sees them.** Changes to
  `AGENTS.md` or the shared instructions require a commit, a push with GitHub,
  and a branch based on the updated default branch.
- **`platform-workspace` uses mount mode.** Its repositories are read-only and
  its `.local/` key material is hidden, but it does not yet use agent-owned
  clones.

## Lessons learned

These shaped the design and are easy to get wrong again:

- The Dev Container CLI adds a default writable workspace mount unless
  `workspaceMount` is set explicitly.
- Podman copies existing content into a tmpfs unless `notmpcopyup` is set, and
  creates tmpfs mounts read-only for unprivileged users unless a mode is set.
- Shared caches and build output between environments let an agent plant code
  that runs elsewhere, invisibly to `git diff`.
- Git hooks and configuration such as `core.fsmonitor` execute on the host.
- A tool may contact more hosts than documented; the proxy log reveals them.
- In repository rulesets, restricting updates to the default branch also
  blocks merging, except by bypass.

## Open questions

- **Agent read access to the kind cluster.** Read-only RBAC, a short-lived
  token, and a route through the proxy were outlined, but not decided. It
  would refine ADR-0008.
- **How much agent help to use in `platform-workspace`,** and whether it should
  adopt agent-owned clones.
- **Starting the agent container without the Dev Container CLI,** with
  `podman run`, to regain SELinux confinement. A separate decision.
- **Automatic GitHub token renewal** with a host process, instead of manual
  renewal.
- **CI for pull requests,** following the rule that pull request workflows
  hold no secrets, once workspaces have workflows.
- **Automatically allocated host ports** for developer containers, replacing
  `FRONTEND_HOST_PORT`, so that several workspaces can run at once.

## Next steps

Coding agents:

1. Exercise a template update end to end: tag a small template change and
   merge it into `example-app` with `task template:update`.
2. Note in `platform-workspace` that `agent.sh` is owned by the template and
   copied from it.
3. Upgrade OpenCode to 2.x, evaluated in a separate clone of `example-app`.
4. Decide on the open questions above as needs arise.

Platform:

1. Close the key-material gaps: back up the Sealed Secrets keys at the right
   time, encrypt the backup, keep a persistent root CA, and practice key
   rotation.
2. Add CI, starting with `shellcheck` and manifest validation.
3. Let GitOps manage the platform components.
4. Close the image delivery loop into `local-environments`.
5. Add a log backend, move to `*.localhost`, evaluate Renovate, and move host
   scripts in `local-platform` to `host/`.

Applications:

1. Continue developing `example-app`, using the agent workflow for real tasks.

## Conventions

- Commit messages follow Conventional Commits, on one line.
- History is linear, except for template updates, which are merge commits.
- Accepted ADRs are not rewritten; facts discovered later are added as dated
  addenda.
- Changes to template-owned files are made in the template and validated in
  `example-app`.
