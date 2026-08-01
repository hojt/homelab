# ADR-0003: Repository Self-Containment

**Status:** Accepted

## Context

The homelab consists of multiple independent repositories.

Repositories should be easy to discover, clone, understand, and develop in
without requiring detailed knowledge of the rest of the project.

A consistent repository structure also reduces the cognitive load when moving
between repositories.

## Decision

Each repository is self-contained.

A repository is expected to provide everything required to understand, build,
and develop that repository, including:

- a README describing its purpose
- a Dev Container
- a `dev.sh` entrypoint
- a `Taskfile.yaml` exposing common development tasks

Repository-specific tooling and automation belong to the repository itself
rather than depending on external scripts or shared infrastructure.

Shared conventions should be preferred over shared implementation.

## Consequences

Repositories can be cloned, explored, and developed independently.

Developers encounter a consistent workflow across repositories.

Changes to one repository do not require changes to another repository's
development tooling.

Cross-repository consistency is maintained through documentation and shared
architectural principles rather than shared scripts.
