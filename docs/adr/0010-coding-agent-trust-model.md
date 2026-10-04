# ADR-0010: Coding Agent Trust Model

**Status:** Accepted

**Refines:** [ADR-0007](0007-application-workspace-structure.md), [ADR-0008](0008-platform-development-workspace.md)

## Context

ADR-0007 and ADR-0008 give coding agents access to application and platform
workspaces. ADR-0008 separates the developer environment from the
coding-agent environment: the developer environment may receive host
container-runtime and Kubernetes access, while the coding-agent environment
does not receive host sockets, Kubernetes credentials, host Git
configuration, or host networking. Containment is verified before the agent
starts. ADR-0008 and ADR-0009 also establish that the human developer owns
staging, commits, tags, and pushes.

These decisions remove the direct capability channels from the coding agent.
They do not yet define how far the agent can reach indirectly:

- Workspaces are bind-mounted read-write, so every change the agent makes
  appears on the host.
- Some workspace content is executed or interpreted outside the coding-agent
  environment: host entrypoints, host scripts, Dev Container definitions, and
  Git metadata such as hooks and configuration.
- Application code, tests, and build configuration written by the agent are
  executed in the developer environment, which holds the host and cluster
  capabilities the agent environment was designed to lack.
- Writable state, such as dependency caches, may be shared between
  environments.
- Outbound network access is unrestricted, so host services, the local
  container registry, the local network, and arbitrary internet destinations
  may be reachable.
- Credentials the agent needs, and other secret material present in the
  workspace, are readable inside the coding-agent environment.

Coding agents act on untrusted input, including repository content,
dependencies, and web content. They may be manipulated through prompt
injection or simply make mistakes. The agent must therefore be treated as an
untrusted actor, and its reach must be bounded by more than the absence of
direct capability channels.

## Decision

The homelab distinguishes the following trust zones:

| Zone                  | Trust     | Description                                                                              |
| --------------------- | --------- | ---------------------------------------------------------------------------------------- |
| Host                  | Trusted   | Developer workstation. Runs host entrypoints and host scripts, some with elevated privileges. |
| Developer environment | Trusted   | Human-driven Dev Container. May operate the local platform as described by ADR-0008.       |
| Platform              | Trusted   | The Kubernetes platform, its container registry, and GitOps-managed environments.          |
| Coding-agent environment | Untrusted | Executes coding agents and the commands they run.                                     |
| External services     | External  | Model providers and package registries required by the agent.                            |

Agent output is untrusted until the human developer has reviewed it. Content
produced in the coding-agent environment reaches a more trusted zone only
through a deliberate human action.

The coding-agent environment follows these principles:

### Nothing executed in a more trusted zone is writable by the agent

Host entrypoints, host scripts, Dev Container definitions, Git metadata, and
the agent's own configuration and permission settings are read-only inside
the coding-agent environment. This applies to every repository mounted into a
workspace.

Content executed or interpreted on the host is kept in well-defined locations
so that it can be protected as a group rather than file by file.

### The agent holds no publishing capability

The coding-agent environment has no credentials for Git remotes, container
registries, or the platform. This makes the human ownership of commits,
tags, and pushes established by ADR-0008 and ADR-0009 an enforced property
rather than a convention.

### The agent cannot reach platform or host services

Outbound network access from the coding-agent environment is denied by
default. Only destinations required for the agent's work, such as the model
provider and package registries, are allowed. The host, the local registry,
the Kubernetes API, and the local network are not reachable.

### The agent shares no writable state with more trusted zones

Caches, volumes, and other writable state used by the coding-agent
environment are not shared with the developer environment or the host.

### The agent only sees the secrets it needs

The only secrets available in the coding-agent environment are the
credentials the agent requires to operate. These credentials are dedicated
to agent use, revocable, and limited in cost where the provider supports it.

Other secret material in a workspace, such as backups of encryption key
material, is not visible inside the coding-agent environment.

### The coding-agent environment runs with least privilege

The environment runs rootless, without privilege escalation, with a minimal
set of capabilities, with host mandatory access control preserved, and with
resource limits.

### Agent-written code is reviewed before it runs with more privilege

Application code, tests, and build configuration written by the agent are
reviewed before they are executed in the developer environment. The
developer environment receives only the host and cluster capabilities that
the current task requires.

### Containment is verified and fails closed

The containment verification required by ADR-0008 covers the properties
above. If verification fails, the agent is not started. The verification is
trustworthy only because the entrypoint performing it is itself protected
from the agent.

### The model is independent of agent tooling

These principles apply to every coding agent run in a workspace, regardless
of which tool or model provider is used.

## Consequences

Each hardening measure can be traced to an explicit trust boundary, and new
agent tooling can be evaluated against the same model.

Host entrypoints remain part of each self-contained repository as required
by ADR-0003. They are protected through read-only access inside the
coding-agent environment rather than by moving them out of the repository.

Protected paths and allowed network destinations must be maintained as
workspaces evolve. New host scripts or new external dependencies require an
explicit change to the coding-agent environment.

The platform workspace requires particular care. It mounts several
repositories, including host scripts that run with elevated privileges and
local backups of encryption key material.

Some risks are reduced but not eliminated:

- Agent credentials can still be exfiltrated through allowed destinations.
  Dedicated, revocable, and cost-limited credentials limit the impact.
- Review before execution in the developer environment depends on human
  diligence. The developer environment's own capabilities bound the impact
  of anything that passes review unnoticed.
- Isolation depends on the host kernel and container runtime. Their
  vulnerabilities are outside the scope of this decision.
- The Dev Container CLI always disables SELinux labeling when running
  containers with Podman. While the coding-agent environment is started
  through the Dev Container CLI, host mandatory access control cannot be
  preserved, and containment relies on the remaining layers. Starting the
  coding-agent environment without the Dev Container CLI would remove this
  constraint, but would be a separate decision.

Existing workspaces adopt these principles through separate, validated
changes. This decision does not claim that existing workspaces already
satisfy them.
