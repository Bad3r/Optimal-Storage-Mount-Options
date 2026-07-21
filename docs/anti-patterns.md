# Rejected Storage Defaults and Anti-Patterns

[Project home](../README.md) | [Documentation index](README.md) | [IEEE references](references.md)

| Rejected advice | Reason | Replacement |
| --- | --- | --- |
| Ext4 for every workload | Maturity alone does not provide checksums, snapshots, compression, or managed multi-disk recovery | Select from the [decision matrix](decision-guide.md#decision-matrix) |
| `noatime,nodiratime` everywhere | Redundant and can break access-time consumers | Keep default `relatime`; measure exceptions |
| `lazytime` everywhere | Delays persistence of inode timestamps | Use only for a measured workload that accepts the crash behavior |
| `barrier=1` everywhere | Already automatic where supported; obsolete on XFS | Keep safe defaults and never use `nobarrier` generically |
| Ext4 `data=ordered` on hard disks only | It is Ext4's default on every block-device type | Omit it |
| Ext4 `commit=30`, `60`, or `300` | Enlarges the crash-loss window | Keep the five-second Ext4 default |
| Btrfs `compress-force=zstd:3` | Wastes CPU on incompressible files; upstream no longer recommends forcing | Use the [Btrfs compression policy](filesystems/btrfs.md) |
| Btrfs `ssd,space_cache=v2,discard=async` | Current detection and defaults already select these behaviors when appropriate | Omit them unless supporting an older measured target |
| Continuous `discard` on every SSD | Can create latency and leaks allocation through encryption | Use the [filesystem-specific TRIM policy](mount-options-and-trim.md#trim-and-discard-policy) |
| `fstrim.service` as the scheduled unit | It is the one-shot worker, not the scheduler | Enable `fstrim.timer` |
| Disable flushes or barriers on Gen5 NVMe | Link speed is unrelated to power-loss ordering | Keep flush and FUA semantics |
| Force 4 KiB namespace or dm-crypt sectors | The three block-size layers are independent; wrong topology can corrupt partial sectors | Inspect and keep automatic defaults |
| Tune scheduler, queues, read-ahead, or dm-crypt workqueues by drive class | Synthetic gains can worsen fairness, thermals, or real latency | Benchmark the complete encrypted workload |
| Btrfs RAID5/6 for production | Official status still identifies severe known problems | RAID1, RAID1C3, RAID1C4, RAID10, or another storage design |
| Bcachefs as a universal 2026 production default | Out-of-tree DKMS and recovery path remain fast-moving | Follow the [evaluation-only profile](filesystems/bcachefs.md) |
| exFAT as a backup or Linux data filesystem | No journal, Unix permissions, or user-data checksums | Ext4 plus LUKS2 for Linux backups; exFAT only for exchange |
| A fixed 8 GiB tmpfs for SSD longevity | Consumes volatile memory or swap and changes failure semantics | Use distribution defaults unless the workload needs tmpfs |
| Rebuild every initramfs after any fstab edit | Most data-mount changes are not early-boot inputs | Rebuild only for distribution-specific early-boot changes |
| Snapshots or RAID as backup | Same-system deletion, compromise, controller failure, and operator error remain | Independent encrypted, restore-tested copies |

The corresponding safe defaults and exceptions are documented in
[mount options and TRIM](mount-options-and-trim.md),
[encryption architecture](encryption.md), and the
[filesystem profile index](filesystems/README.md).
