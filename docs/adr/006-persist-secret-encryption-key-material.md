# Persist secret encryption key material outside disposable clusters

## Context

Local Kubernetes clusters are intentionally disposable and may be
destroyed and recreated frequently.

GitOps-managed secrets must remain recoverable across cluster
recreation without storing plaintext secrets or private key material
in Git.

Secret encryption mechanisms such as Sealed Secrets depend on
cryptographic key material held by the cluster. If that key material
is lost when the cluster is destroyed, existing encrypted secrets
can no longer be recovered.

## Decision

Cryptographic key material required to decrypt GitOps-managed secrets
must have a lifecycle independent of disposable Kubernetes clusters.

For Sealed Secrets, controller-generated sealing keys will be backed
up to secure storage outside the cluster and restored during disaster
recovery or cluster recreation as required.

Private key material must not be stored in Git.

Automatic key renewal may remain enabled. Backups must therefore
include the complete set of active and historical sealing keys needed
to decrypt existing SealedSecret resources.

Backup and restore procedures must be testable and periodically
verified through cluster recreation.

## Consequences

- Kubernetes clusters remain disposable.
- Encrypted secret manifests can remain safely versioned in Git.
- Secret encryption key material becomes persistent platform state.
- Backup of key material becomes part of platform operations.
- Cluster bootstrap must account for restoration of existing key
  material before encrypted secrets are expected to reconcile.
- Loss of both the cluster and its key backup may make encrypted
  secrets unrecoverable.
- Key backup storage requires stronger protection than the Git
  repositories themselves.
