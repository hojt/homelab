# Homelab

A Kubernetes-based homelab built incrementally using GitOps principles.

This repository owns the architectural documentation and decisions for the
homelab project. Implementation is intentionally split across multiple
repositories, while this repository captures the overall vision, design
principles, and architectural decisions.


## Purpose

The primary goal of this project is learning.

This project is primarily a learning journey rather than a production platform.

The homelab serves as a practical platform for exploring modern software
development and platform engineering practices, including topics such as:

- DevOps
- Platform Engineering
- Kubernetes
- GitOps
- Containers
- Cloud-native applications
- Infrastructure as Code
- Observability

The platform itself is not the end goal. It is the vehicle used to gain
practical experience, evaluate technologies, and understand architectural
trade-offs through hands-on implementation.


## Vision

Build a modern homelab while continuously developing practical
knowledge of modern platform engineering and cloud-native technologies.

The project evolves through small, working increments. Simplicity is preferred
over completeness, and abstractions are introduced only when they solve a real
problem.


## Design Principles

### MVP first

Develop the platform through small, independently valuable increments.

Avoid building infrastructure or abstractions before they are needed.


### GitOps

Git is the desired source of truth.

Manual operations are acceptable during bootstrap, but the long-term goal is
that the platform continuously reconciles its desired state from Git.


### Portability

Prefer open standards and portable tooling over vendor-specific solutions.

Avoid unnecessary dependencies on a specific operating system, editor, IDE,
cloud provider, or proprietary platform whenever practical.

Repositories should remain usable across different developer workstations and
execution environments.


### Disposability

Local clusters are treated as disposable infrastructure. Persistent platform 
state required for recovery, including cryptographic key material, is backed 
up independently of the cluster and is never stored unencrypted in Git.


### Kubernetes-first

Applications should be designed to run on Kubernetes.

The local development experience should resemble the deployment environment
whenever practical.


### Separation of responsibilities

Each repository owns a single responsibility.

Platform lifecycle, environment configuration, and application source code are
kept separate.


### Developer experience

Development should be reproducible and consistent across repositories.

Every repository provides:

- a Dev Container
- a `dev.sh` entrypoint
- a `Taskfile.yml` exposing the repository's common development tasks

Personal editor configuration and workflow remain independent of the project.


### Host responsibilities

The developer's host machine is intentionally kept lightweight.

The host is responsible for:

- Git
- Podman
- Dev Container CLI
- tmux
- a terminal emulator
- a text editor

Repository-specific tooling should execute inside the Dev Container whenever
practical.


### Keep it boring

Prefer well-understood technologies.

Delay abstractions until experience demonstrates that they are needed.


## Repository Overview

| Repository | Purpose |
|------------|---------|
| `local-platform` | Bootstrap and operate the local Kubernetes platform. |
| `local-environments` | Own GitOps desired state for Kubernetes environments. |
| `example-app` | Example application workspace used to validate application development, deployment boundaries, and agentic coding workflows. |

Application workspaces may contain multiple independently buildable and
deployable components, such as frontends, backend services, and workers.
Components remain together while they normally evolve as one application and
may be split into separate repositories when a concrete need for independent
ownership or evolution emerges.

Additional repositories may be introduced as the platform evolves.


## Architecture Decision Records

Architectural decisions are documented as Architecture Decision Records (ADRs).

The ADRs describe **why** decisions were made rather than implementation
details.

See [`docs/adr/index.md`](./docs/adr/index.md) for an index of ADRs and [`docs/adr/`](./docs/adr/) for details.


## Status

This project is under active development.

Architecture evolves incrementally, and decisions are documented once they have
proven themselves in practice.
