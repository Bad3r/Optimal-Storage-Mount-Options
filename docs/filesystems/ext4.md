# Ext4: Conservative and Simple Default

[Project overview](../../README.md) | [Documentation](../README.md) |
[Filesystem index](README.md) | [References](../references.md)

Use Ext4 for conservative server roots, virtual-machine roots, uncomplicated
data volumes, and Linux-only external backup disks. It journals metadata and
has broad recovery tooling, but it does not checksum ordinary file contents or
provide native snapshots [19]-[21]. Encryption is LUKS2 below Ext4.

Use current `mke2fs` defaults instead of hardcoding feature bits, inode ratios,
or geometry:

```sh
sudo mkfs.ext4 -L <label> /dev/mapper/<mapping>
```

If selective fscrypt directories are required, create the filesystem with the
feature explicitly enabled:

```sh
sudo mkfs.ext4 -O encrypt -L <label> /dev/mapper/<mapping>
```

Root filesystem:

```fstab
UUID=<EXT4-UUID>  /  ext4  defaults,errors=remount-ro  0  1
```

Ordinary data filesystem:

```fstab
UUID=<EXT4-UUID>  /srv/data  ext4  defaults,errors=remount-ro  0  2
```

External disk on a systemd host:

```fstab
UUID=<EXT4-UUID>  /srv/backup  ext4  defaults,errors=remount-ro,nofail,x-systemd.automount,nodev,nosuid  0  2
```

Use normal Unix ownership and modes inside Ext4. Add `noexec` to a removable
or content-only volume only when software execution is not intended.

Do not add `data=ordered`, `barrier=1`, or `commit=5`: ordered data mode,
barriers, and a five-second commit interval are already defaults [19]. Do not
increase the commit interval to 30, 60, or 300 seconds. Ext4 metadata checksums
protect filesystem structures, not ordinary file data [20]. Backup software
must provide versioning and content verification.
