# ADR-0009: Application Workspace Composition

**Status:** Accepted

**Refines:** [ADR-0007](./0007-application-workspace-structure.md)

## Context

ADR-0007 establishes cohesive application workspaces and keeps components
together while they normally evolve as one application. It also allows
components to move into independent repositories when ownership, reuse, or
development lifecycle creates a concrete need.

Application workspaces need a consistent structure whether their implementation
is owned by one repository or composed from several repositories. Component
location should not depend on component type or repository ownership.

ADR-0008 provides a precedent for separating development workspaces from Git
repositories. Its use of sibling checkouts and individual bind mounts serves
platform development; application workspaces do not currently need that
composition mechanism.

## Decision

Application workspaces place solution components under `components/`, with a
named directory for each component. Components may include frontends, backend
services, workers, or shared libraries. A component directory does not imply
a separate repository or deployment unit.

Ordinary directories tracked by the workspace repository are the default.
Components remain together when they normally evolve and are owned together,
as described in ADR-0007.

When a component requires an independent repository, it is included as a Git
submodule under `components/`. The workspace records the repository location
and a specific commit. Ordinary directories and submodules may coexist in the
same workspace. A workspace may also compose only independently owned
component repositories.

Separate bind mounts of component repositories are not part of this application
workspace convention. Mounting the complete workspace into its Dev Container
remains normal development-environment plumbing. ADR-0008 and the platform
workspace composition model are unchanged.

Component source, tests, and build automation remain with their owning
repository. Independently owned repositories remain usable on their own under
ADR-0003. The workspace owns its development environment, entrypoints, agent
instructions, solution-level documentation, and automation or tests that span
components. Workspace commands delegate component operations to component
automation rather than duplicating it.

The directory convention does not impose one release lifecycle on all
components. Components owned by the workspace may share an application version;
independent repositories retain their own versioning. Submodule references
record a source composition, not the deployed environment state.

The responsibility boundaries of ADR-0005 remain unchanged: application
repositories build and publish artifacts, while environment repositories own
the desired deployment state. Application development does not require edit
access to platform or environment implementation repositories.

## Consequences

One workspace convention supports both a cohesive application repository and
an application assembled from independent repositories, without requiring
separate workspace models for each case.

Submodules make the selected source commits explicit. Reproducing that source
composition requires access to the referenced repositories and commits; it
does not by itself guarantee reproducible builds or a running environment.

Changes to independent repositories require separate review, commits, and
publication before updated references are published in the workspace. There
are no atomic commits across repositories. The human developer continues to
own staging, commits, tags, and pushes in every repository.

Submodule initialization, branch handling for development, and reference updates
must be documented by workspaces that use them. Agents must respect repository
boundaries and the instructions applicable to each component.

Moving a component into its own repository can preserve its workspace path,
but still requires an explicit migration and review of versioning and tooling.

Existing workspaces adopt the convention through separate, validated changes.
This decision does not claim that `example-app` has already migrated, and does
not require a generic workspace generator or automatic submodule updates.
