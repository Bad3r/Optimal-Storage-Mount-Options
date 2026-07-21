# Documentation Index

[Project home](../README.md) | [IEEE references](references.md)

**Research cutoff:** July 21, 2026

The Linux Storage, Filesystem, Partitioning, and Encryption Guide separates
decisions, hardware, encryption, filesystem profiles, mount policy, deployment,
and operations so each subject can be reviewed without editing an unrelated
part of the project.

Inline citation numbers in every module resolve to the shared
[IEEE reference registry](references.md). Reference numbers remain stable when
the documentation is reorganized.

## Start here

| Need | Read |
| --- | --- |
| Choose a complete storage stack | [Decision guide](decision-guide.md) |
| Understand PCIe 5.0, M.2, or NVMe implications | [NVMe hardware guide](nvme.md) |
| Design encryption, key recovery, discard, or swap | [Encryption architecture](encryption.md) |
| Select a filesystem | [Filesystem profile index](filesystems/README.md) |
| Choose fstab options or TRIM behavior | [Mount options and TRIM](mount-options-and-trim.md) |
| Build or migrate a storage stack | [Deployment procedure](deployment.md) |
| Establish monitoring, scrub, backup, and recovery | [Operations and recovery](operations-and-recovery.md) |
| Check advice that this project rejects | [Anti-patterns](anti-patterns.md) |
| Audit the research | [IEEE references](references.md) |

## Filesystem profiles

| Profile | Selected use |
| --- | --- |
| [Btrfs](filesystems/btrfs.md) | Linux laptops and workstations |
| [Ext4](filesystems/ext4.md) | Conservative roots, simple data volumes, and Linux backup disks |
| [XFS](filesystems/xfs.md) | Databases, VM images, containers, and sustained parallel writes |
| [OpenZFS](filesystems/openzfs.md) | Managed NAS and storage servers |
| [F2FS](filesystems/f2fs.md) | Measured eMMC, UFS, or SD workloads |
| [Portable and special-purpose formats](filesystems/portable-and-special-purpose.md) | exFAT, FAT32, NTFS, VeraCrypt, and tmpfs |
| [bcachefs](filesystems/bcachefs.md) | Research and opt-in evaluation only |

## Suggested reading paths

### New Linux workstation

1. Read the [decision guide](decision-guide.md).
2. Review [NVMe hardware behavior](nvme.md).
3. Build the [LUKS2 architecture](encryption.md).
4. Apply the [Btrfs profile](filesystems/btrfs.md).
5. Confirm [mount and TRIM policy](mount-options-and-trim.md).
6. Follow the [deployment](deployment.md) and
   [operations](operations-and-recovery.md) checklists.

### Managed storage server

1. Read the [decision guide](decision-guide.md).
2. Select the [OpenZFS profile](filesystems/openzfs.md), or select
   [Ext4](filesystems/ext4.md) or [XFS](filesystems/xfs.md) over conventional
   RAID when that recovery model is preferred.
3. Resolve the [encryption boundary](encryption.md).
4. Follow the [deployment](deployment.md) and
   [operations](operations-and-recovery.md) guides.

### Removable media

1. Choose Linux-only or cross-platform behavior in the
   [decision guide](decision-guide.md).
2. Apply the [portable-media profile](filesystems/portable-and-special-purpose.md).
3. Resolve LUKS2, VeraCrypt, and discard leakage in the
   [encryption guide](encryption.md).

## Maintenance conventions

- Keep a recommendation in the module that owns the behavior.
- Keep the root README and this index short. Link to details instead of copying
  them.
- Add new primary sources to [references.md](references.md) using the next
  sequential IEEE number. Do not renumber existing citations without updating
  every module.
- Update the research cutoff only after revalidating every time-sensitive
  recommendation affected by the change.
- Keep hardware generation, encryption, filesystem, and mount-option decisions
  separate. A new SSD generation does not automatically justify a filesystem
  tuning change.
