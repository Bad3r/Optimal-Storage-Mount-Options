# Storage Decision Guide

[Project home](../README.md) | [Documentation index](README.md) | [References](references.md)

## Non-negotiable rules

1. **Encryption is part of the design, not a later mount option.** Linux-native
   filesystems containing private data go inside LUKS2. OpenZFS uses native
   encrypted datasets when its visible pool metadata is acceptable. Portable
   cross-platform confidential media uses VeraCrypt.

2. **Snapshots and RAID are not backups.** Keep an independently encrypted
   backup on separate media or a separate system, keep at least one copy
   unreachable through routine host credentials, and test restoration.

3. **Keep write ordering enabled.** Never disable barriers, cache flushes, or
   force-unit-access behavior for benchmark results. PCIe 5.0 does not make a
   volatile controller cache power-safe. Applications still need correct
   `fsync()` or `fdatasync()` behavior for durable transactions [1], [4].

4. **Keep filesystem defaults unless this guide names an exception.** Current
   kernels and filesystem tools already select safe geometry, SSD detection,
   journal behavior, free-space caches, and queue handling. A measured workload
   may justify a deviation, but the deviation must record its failure cost.

5. **Do not lengthen journal commit intervals to reduce SSD writes.** This
   increases the amount of recent work that can be lost after a crash. It does
   not provide a meaningful NVMe endurance policy.

6. **Checksums, redundancy, and encryption are separate properties.** A
   checksum can detect accidental corruption. A redundant verified copy can
   repair it. Encryption protects confidentiality while locked. Ordinary
   dm-crypt does not authenticate every writable sector [12], [14].

7. **Use persistent device identities.** Put filesystem UUIDs in `/etc/fstab`,
   LUKS UUIDs in `/etc/crypttab`, and persistent `/dev/disk/by-id/` paths in
   destructive administrative commands. Never rely on `/dev/sdX` or namespace
   numbering remaining stable.

8. **Capacity is part of reliability.** Alert at 80 percent allocation and act
   before 90 percent. Copy-on-write filesystems need free space for new extents,
   snapshots, repair, and maintenance. The percentages are this guide's
   operational policy, not filesystem limits.

## Decision matrix

| Use case | Selected stack | Why it wins | Main cost |
| --- | --- | --- | --- |
| Laptop or workstation root and home | GPT, LUKS2, Btrfs | Checksummed data, transparent compression, snapshots, reflinks, scrub, and in-kernel support | Requires snapshot and free-space discipline |
| Conservative server or virtual-machine root | GPT, LUKS2, Ext4 | Broad support, simple recovery model, mature online growth and offline shrink | No user-data checksums, snapshots, or compression |
| Database, VM image, build cache, container data, or large parallel file workload | GPT, LUKS2, XFS | Strong concurrent I/O behavior, allocation groups, reflinks, mature online growth | No user-data checksums and no general routine shrink workflow |
| Simple Linux data or backup disk | GPT, LUKS2, Ext4 | Lowest operational complexity and broad Linux recovery tooling | Backup software must provide content verification and version history |
| Two-disk personal mirror with snapshot workflows | LUKS2 on each member, Btrfs RAID1 for data and metadata | Checksummed redundant copies, scrub repair, snapshots, and send/receive | Each member must be unlocked; RAID5/6 is excluded |
| Conventional application-server mirror | Members, md RAID1, LUKS2, then Ext4 or XFS | Familiar block-RAID failure boundary and one encryption mapping | Filesystem cannot identify which mirror copy has valid user data |
| Managed NAS or storage server | OpenZFS mirrors or RAIDZ2, native encrypted datasets | End-to-end checksums, repair from redundancy, snapshots, datasets, and raw encrypted replication | Out-of-tree Linux module and ZFS-specific administration |
| eMMC, UFS, or SD workload proven by testing | GPT, LUKS2, F2FS | Flash-aware log-structured allocation and hot/cold separation | Garbage-collection latency and a smaller recovery ecosystem |
| Confidential cross-platform removable media | VeraCrypt standard volume, exFAT inside | Current Linux, Windows, and macOS access without exposing plaintext at rest | No native Unix permissions or journal; every host needs VeraCrypt |
| Non-confidential cross-platform exchange | exFAT | Broad platform and large-file interoperability | No journal, Unix ownership, or user-data checksum; never the only copy |
| Windows-owned volume occasionally read by Linux | NTFS, created and maintained by Windows | Preserves Windows semantics and repair ownership | Not selected for a Linux root or Linux-owned application volume |
| EFI System Partition or firmware-required media | FAT32 | Required interoperability | Unencrypted, no journal, and no Unix security model |
| Read-only appliance or verified OS image | EROFS or SquashFS plus dm-verity | Compact immutable image with cryptographic block verification [15], [48] | Image must be rebuilt to change it; not a writable data filesystem |

### Feature boundary

| Filesystem | Ordinary file-data checksums | Repair from redundant copy | Snapshots | Compression | Encryption policy in this guide |
| --- | --- | --- | --- | --- | --- |
| Btrfs | Yes, unless NOCOW disables them | Yes with a redundant profile | Native | Zstd | LUKS2 below Btrfs |
| Ext4 | No | No | External layer only | No | LUKS2 below Ext4 |
| XFS | No; metadata has CRCs | No | External layer only; reflinks are supported | No | LUKS2 below XFS |
| F2FS | No general user-data checksum | No | No general snapshot facility | Specialized per-file compression | LUKS2 below F2FS |
| OpenZFS | Yes | Yes with redundant vdevs | Native | LZ4 or Zstd | Native encrypted datasets by default |
| exFAT and FAT32 | No | No | No | No | VeraCrypt container when portability is required |
