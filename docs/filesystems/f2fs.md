# F2FS: Measured Flash-Specific Exception

[Project overview](../../README.md) | [Documentation](../README.md) |
[Filesystem index](README.md) | [References](../references.md)

F2FS was designed for NAND-backed storage and separates hot and cold data, but
cleaning can create long latency outliers [30]. Select it only after testing the
real workload at realistic fullness and after a power-loss and recovery test.
Do not select it merely because an SSD is NVMe or PCIe 5.0.

```sh
sudo mkfs.f2fs -O extra_attr,inode_checksum,sb_checksum \
  -l <label> /dev/mapper/<mapping>
```

If selective fscrypt directories are required, include `encrypt` in that
creation-time feature list:

```sh
sudo mkfs.f2fs -O encrypt,extra_attr,inode_checksum,sb_checksum \
  -l <label> /dev/mapper/<mapping>
```

```fstab
UUID=<F2FS-UUID>  /srv/flash  f2fs  defaults,errors=remount-ro  0  0
```

The explicit error policy overrides the current `errors=continue` default.
Keep barriers, checkpoints, and garbage collection enabled. Do not apply F2FS
compression generically: it is a specialized per-file or directory policy with
unusual space-accounting behavior, intended mainly for compressible,
mostly-write-once files. The selected inode and superblock checksums protect
metadata only; they do not turn F2FS into an end-to-end user-data-checksumming
filesystem [30], [52].

Use LUKS2 for full-volume confidentiality. Use fscrypt only when selective
directory keys are required.
