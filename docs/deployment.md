# Storage Stack Deployment Procedure

[Project home](../README.md) | [Documentation index](README.md) | [IEEE references](references.md)

## 1. Record the current system

Before changing storage, save the output of:

```sh
lsblk -e7 -o NAME,PATH,TYPE,SIZE,FSTYPE,FSVER,LABEL,UUID,PARTUUID,MOUNTPOINTS
findmnt -o TARGET,SOURCE,FSTYPE,VFS-OPTIONS,FS-OPTIONS
sudo blkid
sudo cryptsetup luksDump /dev/disk/by-id/<existing-luks-device>
```

Also record firmware settings, boot mode, partition layout, LUKS UUIDs,
enrollment method, filesystem UUIDs, mount points, swap or resume target, and
the commands needed to rebuild the initramfs and boot entries.

## 2. Select the row, not individual favorite features

Choose one complete stack from the [decision matrix](decision-guide.md#decision-matrix).
Do not combine COW snapshots, external RAID, multiple encryption systems, and
exotic tuning without assigning ownership for recovery at every layer.

## 3. Prove the backup

Restore representative files and metadata to a separate location. Verify that
the backup includes permissions, ACLs, extended attributes, sparse files,
hard links, database-consistent exports, and encryption recovery material where
applicable.

## 4. Resolve the destructive target

Use `/dev/disk/by-id/` and verify model, serial, size, partitions, and active
mounts immediately before partitioning, `luksFormat`, or `mkfs`. Unmount and
close only the exact target. Do not use a broad wildcard.

## 5. Create the selected encryption and filesystem layers

Use the [LUKS2 baseline](encryption.md#luks2-baseline) and exactly one
[filesystem profile](filesystems/README.md). Do not add `--force`, `-F`, or
equivalent safety-bypass flags to generic instructions. After creation, obtain
identifiers with:

```sh
lsblk -f
sudo blkid
sudo cryptsetup luksUUID /dev/disk/by-id/<exact-device>-part2
```

## 6. Validate before relying on the volume

On a maintenance boot or otherwise safe window:

```sh
findmnt --verify --verbose
sudo mount --all --verbose
findmnt -o TARGET,SOURCE,FSTYPE,VFS-OPTIONS,FS-OPTIONS
systemctl --failed
```

Write, sync, read, and delete a non-secret test file. Confirm ownership and
security flags as the intended user. If the volume is removable, test absence
at boot. If hibernation is required, test resume after encryption unlock. If a
token unlock is used, also test the recovery credential and rescue media.

## 7. Benchmark the final stack

Benchmark a disposable file on the mounted, encrypted filesystem, not the raw
device. Use representative block sizes, queue depth, read/write mix, dataset
fullness, snapshot count, and application durability settings. Monitor CPU,
temperature, throttling, memory pressure, and latency percentiles. A raw
sequential headline cannot validate the storage policy.

Continue with the [operations and recovery](operations-and-recovery.md)
schedule after deployment.
