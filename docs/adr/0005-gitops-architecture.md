# ADR-0005: GitOps Architecture

**Status:** Accepted

## Context

The homelab aims to adopt GitOps as the operational model for managing the
Kubernetes platform.

The project consists of separate repositories with different responsibilities.
A clear ownership model is required to distinguish between platform bootstrap,
desired environment state, and application source code.

The GitOps architecture should support reproducibility while remaining simple
enough to evolve incrementally.

## Decision

The homelab adopts GitOps as the deployment model for Kubernetes workloads.

Repository responsibilities are divided as follows:

| Repository | Responsibility |
|------------|----------------|
| `local-platform` | Bootstrap and operate the Kubernetes platform, including the GitOps engine. |
| `local-environments` | Define the desired state of Kubernetes environments. |
| Application repositories | Build application artifacts and publish container images. They do not deploy themselves. |

The GitOps engine continuously reconciles the desired state stored in
`local-environments` with the actual state of the Kubernetes cluster.

The initial implementation uses Argo CD as the GitOps engine.

The choice of GitOps engine is considered an implementation detail rather than
an architectural decision.

## Consequences

The desired state of the platform is managed through Git rather than manual
cluster operations.

Application deployment is decoupled from application development.

Platform bootstrap and environment configuration evolve independently.

Alternative GitOps engines may be evaluated in the future without changing the
overall architecture.
