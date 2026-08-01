# ADR-0002: Development Environment Philosophy

**Status:** Accepted

## Context

The homelab consists of multiple independent repositories that should provide a
consistent development experience.

Developers may use different operating systems, editors, and personal
workflows. At the same time, repository-specific tooling should behave
consistently regardless of the host environment.

The project therefore requires a clear separation between the developer's
workstation and the repository's development environment.

## Decision

The developer's host machine is treated as a workstation rather than a
development environment.

The host is expected to provide only a small set of common development tools,
including:

- Git
- Podman
- Dev Container CLI
- tmux
- a terminal emulator
- a text editor

Repository-specific tooling should execute inside the repository's Dev
Container whenever practical.

Personal development preferences, such as editor configuration, shell
configuration, and terminal choice, remain the responsibility of the developer
and are intentionally kept outside the repositories.

## Consequences

Repositories provide a reproducible development environment independent of the
developer's personal workstation.

Repository-specific dependencies do not need to be installed on the host.

Developers remain free to use their preferred editor and workflow while sharing
a common development environment.

Changes to repository tooling affect only the Dev Container and do not require
changes to the developer's workstation.
