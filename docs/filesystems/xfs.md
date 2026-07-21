# XFS: Parallel and Rewrite-Heavy Data

[Project overview](../../README.md) | [Documentation](../README.md) |
[Filesystem index](README.md) | [References](../references.md)

Use XFS for large parallel files, VM images, databases, container data, media
workspaces, and volumes expected primarily to grow. XFS journals metadata and
current V5 filesystems protect metadata with CRCs, but it does not checksum
ordinary file contents [27], [28]. Use LUKS2 below XFS.

Use current `xfsprogs` geometry and CRC/reflink defaults. This guide explicitly
enables large timestamps so newly created inode timestamps are not limited by
the legacy 2038 range. Every boot, rescue, and recovery environment that will
touch the filesystem must support this feature [28]:

```sh
sudo mkfs.xfs -m bigtime=1 -L <label> /dev/mapper/<mapping>
```

```fstab
UUID=<XFS-UUID>  /srv/data  xfs  defaults  0  0
```

Do not add `barrier` or `nobarrier`; those XFS mount options were removed in
Linux 4.19. Do not add continuous `discard`, fixed `allocsize`, manually chosen
log buffers, or RAID stripe geometry without verified topology and a measured
reason [27]. Keep the default mount behavior, and require applications to use
`fsync()` or `fdatasync()` correctly for durable transactions.

Online growth is mature. Current tools have narrowly limited last-allocation-
group shrink support, but this is not a general routine shrink workflow. Plan
XFS capacity on that basis [29]. Run repair only on an unmounted filesystem;
log zeroing can lose recent metadata changes.
