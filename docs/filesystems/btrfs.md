# Btrfs: Workstation and Laptop Default

[Project overview](../../README.md) | [Documentation](../README.md) |
[Filesystem index](README.md) | [References](../references.md)

Use Btrfs for a general Linux workstation because its checksummed data,
snapshots, transparent compression, reflinks, and scrub provide more useful
local recovery and corruption detection than Ext4 without requiring an
out-of-tree kernel module [22]-[25]. Use LUKS2 below it because released Btrfs
does not yet provide production native encryption.

## Layout and mount policy

Create at least these subvolumes:

- `@` for `/`
- `@home` for `/home`
- `@var_log` for `/var/log`, so a root rollback does not erase later logs
- `@snapshots` for locally managed snapshots

Keep application data on `@` unless it needs a different snapshot lifetime.
Nested subvolumes are not included recursively in a parent snapshot, so each
boundary must be intentional. Roll back `@home` only as a separate user-data
decision. Same-device snapshots are not backups.

The selected fast-storage compression policy is Zstd level 1. Upstream
describes levels 1 through 3 as near-real-time choices, while higher levels use
more CPU [23]. Use normal `compress`, not `compress-force`; current Btrfs
compression heuristics are designed to reject incompressible data.

```fstab
UUID=<BTRFS-UUID>  /           btrfs  defaults,subvol=@,compress=zstd:1          0  0
UUID=<BTRFS-UUID>  /home       btrfs  defaults,subvol=@home,compress=zstd:1      0  0
UUID=<BTRFS-UUID>  /var/log    btrfs  defaults,subvol=@var_log,compress=zstd:1   0  0
UUID=<BTRFS-UUID>  /.snapshots btrfs  defaults,subvol=@snapshots,compress=zstd:1 0  0
```

Most Btrfs mount options apply to the entire filesystem, and options from the
first mounted subvolume take effect. Keep filesystem-wide options identical on
every line [22]. Do not add:

- `ssd`: device detection and modern allocation behavior are automatic.
- `ssd_spread`: this specialized, default-off allocation mode needs measured
  evidence on the exact device.
- `space_cache=v2`: it is already the current default.
- `discard=async`: it is automatically selected on supported devices since
  Linux 6.2.
- `compress-force`: upstream does not recommend forcing compression.
- `autodefrag`: it can be unsuitable for databases and can unshare reflinked
  extents.
- `nodatacow`, `nodatasum`, or `nobarrier`: these remove core integrity or
  power-loss protections.
- a longer `commit=` interval: it increases the crash-loss window.

For a latency-sensitive database or VM store, use XFS rather than globally
turning Btrfs into a non-checksumming filesystem. If a measured exception must
use NOCOW, place it in a dedicated empty directory or subvolume and set
`chattr +C` before files are created. Document that this also disables data
checksums, compression, scrub verification, and checksum-guided repair for
those files [22], [25].

## Multi-device Btrfs

For a two-device personal storage set, use LUKS2 on each member and Btrfs RAID1
for both data and metadata. Btrfs RAID1 means two copies on distinct devices,
not fixed mirror pairs. Use RAID10 when at least four devices and the workload
justify striping mirrored copies.

Do not use Btrfs RAID5 or RAID6 for production. The current official status
marks RAID56 unstable, with known severe problems [24]. Scrub monthly:

```sh
sudo btrfs scrub start -Bd /
sudo btrfs filesystem usage /
```

A filesystem-path scrub validates every member device; an explicit device
argument limits the pass to that device. Repair requires a verified redundant
copy. Do not run an unfiltered balance as routine maintenance, and never run
`btrfs check --repair` without specialist recovery direction.
