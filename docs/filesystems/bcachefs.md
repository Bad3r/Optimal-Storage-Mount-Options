# bcachefs: Evaluate, Do Not Standardize

[Project overview](../../README.md) | [Documentation](../README.md) |
[Filesystem index](README.md) | [References](../references.md)

Bcachefs has compelling copy-on-write, checksum, compression, replication, and
authenticated-encryption features. It is not selected for irreplaceable
production data at this research cutoff. Its current Kconfig still labels the
filesystem `EXPERIMENTAL`; it ships a DKMS module after removal from the
mainline kernel in Linux 6.18; full online fsck remains work in progress; and
the July 2026 changelog contains recent fsck, journal, erasure-coding, deadlock,
snapshot-repair, out-of-memory, and encryption fixes [37], [38], [50], [51].

Use it for research or opt-in deployments only, with current packages,
independent tested restores, and willingness to follow fast-moving upstream
recovery guidance. Re-evaluate this decision when in-tree or equivalently
stable distribution support and a complete online checking path are established.

For any non-disposable multi-device evaluation, explicitly set both
`data_replicas` and `metadata_replicas` to at least 2; both default to 1. NOCOW
data is not checksummed or encrypted by native bcachefs encryption and is stored
as plaintext. Keep LUKS below bcachefs if NOCOW may be used. Never mount an
external LVM, zvol, or VM snapshot of an encrypted bcachefs filesystem
read-write because independent instances can reuse encryption nonces.
Bcachefs-native snapshots do not have that external-snapshot problem [49].
