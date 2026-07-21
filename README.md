# Linux Storage, Filesystem, Partitioning, and Encryption Guide

**Research cutoff:** July 21, 2026

An opinionated, decision-complete guide to Linux storage architecture. It
covers filesystem selection, disk partitioning, encryption, PCIe 5.0 NVMe
behavior, mount policy, TRIM, deployment, monitoring, backup, and recovery.

The project chooses a complete storage stack by workload, integrity model,
recovery model, and portability. It does not choose a filesystem from a drive's
advertised sequential speed or apply generic mount-option folklore.

## Default decisions

| Use case | Selected stack |
| --- | --- |
| Linux laptop or workstation | LUKS2 with [Btrfs](docs/filesystems/btrfs.md) |
| Conservative server, virtual-machine root, or simple Linux data disk | LUKS2 with [Ext4](docs/filesystems/ext4.md) |
| Database, VM image, container data, build cache, or sustained parallel writes | LUKS2 with [XFS](docs/filesystems/xfs.md) |
| Managed NAS or storage server | [OpenZFS](docs/filesystems/openzfs.md) with native encrypted datasets |
| Proven eMMC, UFS, or SD workload | LUKS2 with [F2FS](docs/filesystems/f2fs.md) |
| Linux-only external backup disk | LUKS2 with [Ext4](docs/filesystems/ext4.md) |
| Confidential cross-platform removable media | VeraCrypt with exFAT inside |
| Non-confidential cross-platform exchange | exFAT |
| EFI System Partition or firmware-required media | FAT32 |
| Experimental filesystem research | [bcachefs evaluation profile](docs/filesystems/bcachefs.md) |

The [full decision matrix](docs/decision-guide.md#decision-matrix) records the
reason, cost, integrity boundary, and alternatives for each choice. Portable
formats and VeraCrypt are covered in the
[portable and special-purpose profile](docs/filesystems/portable-and-special-purpose.md).

## Core conclusions

- M.2 is a form factor, PCIe is a transport, NVMe is a protocol, and the
  filesystem is a higher software layer. PCIe 5.0 changes bandwidth, thermal
  pressure, and where bottlenecks appear. It does not create a Gen5 mount
  option or remove crash-consistency requirements.
- Encryption is part of the architecture. Linux-native filesystems containing
  private data use LUKS2. OpenZFS uses native encrypted datasets when its
  visible pool metadata is acceptable. Cross-platform confidential media uses
  VeraCrypt.
- Checksums, redundant copies, encryption, snapshots, and backups solve
  different problems. None substitutes for all the others.
- Safe write ordering remains enabled. Barriers, cache flushes, and FUA are not
  disabled for benchmark results.
- Filesystem and cryptsetup defaults remain in place unless a documented,
  measured workload justifies an exception.
- Snapshots and RAID are not backups. The operational policy requires an
  independently encrypted, restore-tested copy.

## Documentation

Start with the [documentation index](docs/README.md), or open the module that
owns the current decision:

### Architecture and hardware

- [Decision guide](docs/decision-guide.md): non-negotiable rules, complete
  use-case matrix, and feature boundaries.
- [PCIe 5.0 NVMe](docs/nvme.md): layer semantics, thermal behavior, health
  inspection, and rejected hardware-class tuning.
- [Encryption architecture](docs/encryption.md): LUKS2, OpenZFS native
  encryption, fscrypt, dm-integrity, dm-verity, TPM2, discard leakage, swap,
  hibernation, and key recovery.

### Filesystem profiles

- [Filesystem index](docs/filesystems/README.md)
- [Btrfs](docs/filesystems/btrfs.md)
- [Ext4](docs/filesystems/ext4.md)
- [XFS](docs/filesystems/xfs.md)
- [OpenZFS](docs/filesystems/openzfs.md)
- [F2FS](docs/filesystems/f2fs.md)
- [Portable and special-purpose formats](docs/filesystems/portable-and-special-purpose.md)
- [bcachefs evaluation policy](docs/filesystems/bcachefs.md)

### Configuration and operations

- [Mount options and TRIM](docs/mount-options-and-trim.md)
- [Deployment procedure](docs/deployment.md)
- [Operations and recovery](docs/operations-and-recovery.md)
- [Rejected defaults and anti-patterns](docs/anti-patterns.md)
- [IEEE reference registry](docs/references.md)

## Safety boundary

Storage-formatting and encryption commands are destructive when pointed at a
real device. Every placeholder must be replaced, the exact persistent device
path must be verified with `lsblk`, and a tested backup must exist before
running `cryptsetup luksFormat`, `mkfs`, partitioning, discard, sanitize, or
repair operations.

Use the [deployment procedure](docs/deployment.md) for the required inventory,
backup proof, persistent-device resolution, validation, and end-to-end
benchmark sequence.

## Research policy

Recommendations rely on current primary or upstream sources, including Linux
kernel documentation, NVM Express and PCI-SIG specifications, cryptsetup and
systemd manuals, filesystem project documentation, Microsoft filesystem
specifications, and vendor hardware documentation.

The shared [IEEE reference registry](docs/references.md) keeps citation numbers
stable across modules. A time-sensitive recommendation must be revalidated
against its cited source before the research cutoff is advanced.
