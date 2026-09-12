# ADR-0007: Application Workspace Structure

**Status:** Accepted

**Refines:** ADR-0001

## Context

The homelab separates platform lifecycle, desired environment state, and
application source code into repositories with explicit responsibilities.

The initial example application was split into separate backend and frontend
repositories. This provides strong component isolation, but it also makes
cross-component application development more fragmented.

A single application feature may require coordinated changes across a frontend,
one or more backend services, shared contracts, tests, and development tooling.
These components may be independently deployable without being independently
developed or owned.

The project also aims to explore agentic coding workflows. Coding agents benefit
from being able to understand and modify the complete application context for a
feature without requiring access to the implementation repositories for the
underlying platform.

Repository boundaries therefore need to reflect cohesive development and
ownership rather than automatically matching deployment boundaries.

## Decision

Applications are initially developed as cohesive application workspaces.

An application workspace is a self-contained application repository and may
contain multiple independently buildable and deployable components, including:

- frontend applications
- backend services
- workers
- shared application packages or contracts
- application-level tests and development tooling

Deployment boundaries do not by themselves require separate repositories.

Components remain in the application workspace while they normally evolve as
part of the same application. A component may be extracted into its own
repository when practical experience demonstrates a need for independent
development, such as:

- independent ownership
- an independent release or development lifecycle
- multiple applications consuming the component
- separate security or access-control requirements
- substantially different evolution or maintenance needs

The initial application workspace is implemented as a normal Git repository.
Repository composition mechanisms such as Git submodules are deferred until a
concrete need for independently versioned component repositories exists.

`example-app` is the reference implementation of this application workspace
model.

The existing platform responsibility boundaries remain unchanged:

| Repository | Responsibility |
|------------|----------------|
| `local-platform` | Bootstrap and operate the Kubernetes platform. |
| `local-environments` | Define desired Kubernetes environment state and workload deployment configuration. |
| Application workspaces | Own application source, tests, build tooling, and published application artifacts. |

Application workspaces consume the platform through documented contracts and
conventions. They do not require edit access to `local-platform` or
`local-environments` in order to develop application features.

Each application workspace remains self-contained according to ADR-0003 and
provides its own development environment and repository-level automation.

## Consequences

Cross-component application features can be developed, tested, reviewed, and
versioned as one coherent change.

Coding agents can operate with application-wide context while remaining outside
platform and environment implementation repositories.

Frontend, backend, worker, and other components can still be built, published,
deployed, scaled, and operated independently.

Application Dev Containers may contain tooling for multiple technology stacks
and may therefore be larger than component-specific development environments.

Application repositories may grow in size and scope as the application evolves.

Extracting a component into a separate repository later requires an explicit
migration, but that complexity is accepted only when independent ownership or
evolution has demonstrated its value.

The project avoids introducing polyrepo coordination mechanisms before there is
a concrete need for them.
