# ADR-0001: Repository Structure

**Status:** Accepted
**Refined by:** ADR-0007

## Context

The homelab consists of several independent but related software projects.

As the platform grows, infrastructure, environments, and applications evolve at
different rates and have different responsibilities.

A single repository would mix concerns and make ownership less clear.

## Decision

The homelab is organized as multiple repositories.

Each repository owns a single primary responsibility.

Current repositories include:

| Repository | Responsibility |
|------------|----------------|
| `local-platform` | Bootstrap and operate the local Kubernetes platform. |
| `local-environments` | Own GitOps desired state for Kubernetes environments. |
| `example-backend` | Example backend application used to validate the platform and development workflow. |
| `example-frontend` | Example frontend application used to validate the platform and development workflow. |

Each repository is independently versioned and should remain usable on its own.

Repository-specific documentation, development environments, and automation
belong to the repository itself.

## Consequences

The boundaries between platform, environments, and applications become explicit.

Repositories can evolve independently while sharing common architectural
principles.

Cross-repository consistency is achieved through shared conventions rather than
a shared codebase.

Developers can work with a single repository without needing to understand the
entire homelab.
