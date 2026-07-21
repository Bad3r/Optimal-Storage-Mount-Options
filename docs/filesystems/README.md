# Filesystem Profiles

[Project overview](../../README.md) | [Documentation](../README.md) |
[Filesystem index](README.md) | [References](../references.md)

Use this index after selecting a workload in the
[decision guide](../decision-guide.md). Filesystem choice follows the workload,
recovery model, portability requirement, and administrative capacity. It does
not follow the PCIe generation or marketing class of the storage device.

| Profile | Selected use |
| --- | --- |
| [Btrfs](btrfs.md) | General Linux workstations, laptops, snapshots, checksummed data, and two-device personal storage |
| [Ext4](ext4.md) | Conservative roots, simple data volumes, virtual-machine roots, and Linux-only backup disks |
| [XFS](xfs.md) | Databases, VM images, containers, media workspaces, large files, and sustained parallel writes |
| [OpenZFS](openzfs.md) | Deliberately administered NAS and multi-disk storage servers |
| [F2FS](f2fs.md) | Measured eMMC, UFS, or SD workloads that justify flash-specific behavior |
| [Portable and special-purpose filesystems](portable-and-special-purpose.md) | exFAT, FAT32, NTFS, VeraCrypt containers, and tmpfs |
| [bcachefs](bcachefs.md) | Research and explicitly opt-in evaluation, not the production default |

Encryption is a separate layer decision. Use LUKS2 below Btrfs, Ext4, XFS, or
F2FS for full-volume confidentiality. OpenZFS normally uses native encrypted
datasets. Confidential portable media uses a current VeraCrypt standard volume
with exFAT inside. See the [encryption guide](../encryption.md) before creating
a filesystem.
