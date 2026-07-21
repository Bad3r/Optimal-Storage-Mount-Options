# Storage Operations and Recovery

[Project home](../README.md) | [Documentation index](README.md) | [IEEE references](references.md)

## Routine schedule

| Interval | Action |
| --- | --- |
| Continuous | Alert on kernel I/O errors, filesystem errors, pool degradation, NVMe critical warnings, media errors, and capacity above 80 percent |
| Weekly | Run distribution `fstrim.timer` where selected; verify backup jobs and off-host copy reachability |
| Monthly | Review NVMe health trends; scrub Btrfs and OpenZFS; inspect `zpool status`; restore a small random backup sample |
| Quarterly | Perform a documented restore test to separate storage; test LUKS recovery credentials and rescue boot path |
| After key or token changes | Refresh and protect the LUKS header backup; retire obsolete copies according to credential policy |
| After kernel, boot, or firmware changes | Verify normal unlock, recovery unlock, Secure Boot state, hibernation if used, and storage health counters |

Scrub verifies checksummed blocks; it does not validate whether a database is
logically consistent or whether the wanted file was deleted. Ext4 and XFS do
not gain user-data verification from a SMART test. Backup tools should verify
stored content cryptographically, and application-aware exports or quiescing
are required for transaction-consistent backups.

The filesystem-specific scrub and health commands remain in the
[filesystem profiles](filesystems/README.md). Device health fields are defined
in the [NVMe hardware guide](nvme.md), and discard scheduling is defined in the
[mount and TRIM guide](mount-options-and-trim.md).

## Failure rules

- Stop writes when the kernel reports I/O or filesystem corruption.
- Preserve logs and current metadata before attempting repair.
- If hardware is failing, image or replace it before repeatedly running a
  filesystem repair tool.
- Run `fsck`, `xfs_repair`, or equivalent writing repairs only against an
  unmounted clone or after preserving the only copy.
- Use `btrfs check` read-only first. Do not use `--repair` without current
  specialist guidance.
- Never restore a LUKS header over the only copy merely to see whether it helps.
  Test a header backup with cryptsetup's detached `--header` option first.
- A lost LUKS volume key, a lost ZFS encryption key, or all invalid redundant
  copies has no filesystem mount-option workaround. Recovery depends on the
  independent backup and key material.

## Recovery ownership

| Failure boundary | Primary recovery owner |
| --- | --- |
| NVMe health or media failure | Device replacement and independent backup |
| LUKS header or credential loss | Protected header backup and independent recovery credential |
| Filesystem structural damage | Filesystem-specific read-only inspection, clone, then repair |
| Checksum mismatch with redundancy | Btrfs or OpenZFS scrub and verified replica |
| Logical deletion or application corruption | Versioned independent backup or application-consistent export |
| Boot-chain or TPM policy change | Rescue media, recovery credential, and signed boot configuration |

Do not cross recovery boundaries casually. A filesystem repair cannot restore a
lost encryption key, SMART cannot validate ordinary file contents, and RAID
cannot recover a deliberately deleted file.
