# OpenZFS: Managed Storage Server

[Project overview](../../README.md) | [Documentation](../README.md) |
[Filesystem index](README.md) | [References](../references.md)

Use OpenZFS for a deliberately administered NAS or storage server where
end-to-end checksums, self-healing redundancy, snapshots, datasets, quotas,
scrubs, and replication are requirements. Keep the distribution kernel and
OpenZFS package on a supported combination because the Linux module is outside
the mainline kernel.

Topology is more important than mount properties [31]:

- Use mirrors for random IOPS, databases, VMs, and easier incremental expansion.
- Use RAIDZ2 for capacity-oriented storage that needs two-device fault
  tolerance.
- Do not create an important nonredundant pool.
- Do not add a single-disk top-level vdev to a redundant pool.
- Give special vdevs redundancy at least equal to the normal data vdevs.
- Do not treat `copies=2` as a replacement for redundant vdevs.

Pool creation is intentionally not reduced to a copy-paste command because a
wrong vdev list is destructive and persistent. Use persistent by-id device
paths. Inspect sector topology before choosing `ashift`; it is immutable per
vdev. `ashift=12` is common for 4 KiB sectors, but PCIe generation and NAND page
size are not evidence for forcing it [34].

For Linux datasets, the general policy is:

- `compression=lz4` for low-overhead general storage.
- `checksum=on` and `sync=standard`, always.
- `dedup=off` unless a measured use case, memory model, and recovery plan prove
  it beneficial.
- `recordsize=128K` for general files. Tune only a dedicated fixed-record
  database or VM dataset after measurement.
- `acltype=posixacl` and `xattr=sa` for normal Linux ACL and xattr behavior.
- Keep at least 10 percent pool space free [34].

Create a new native encryption root, not merely an unencrypted child dataset:

```sh
sudo zfs create \
  -o encryption=on \
  -o keyformat=passphrase \
  -o keylocation=prompt \
  -o compression=lz4 \
  -o acltype=posixacl \
  -o xattr=sa \
  <pool>/private
```

`encryption=on` currently selects AES-256-GCM. Encryption is applied after
compression and must be selected when the encryption-root dataset is created
[32], [33]. It protects file and zvol data, file attributes, ACLs, permissions,
and directory listings. It does not hide pool structure, dataset and snapshot
names, hierarchy, properties, file sizes, holes, or deduplication tables.
Changing a passphrase rewraps the same master key; it does not rotate the data
encryption key, and the prior wrapped master key may remain forensically
recoverable [33]. After suspected key exfiltration, create a new encryption
root with a new master key, copy or send the data into it, and securely retire
the old storage where possible. A passphrase change alone is not remediation.

Use raw encrypted replication to an untrusted receiver:

```sh
sudo zfs send -w <pool>/private@<snapshot> | <protected-transport-to-receiver>
```

The pipeline placeholder is not a literal command. Design the transport,
receiver permissions, retention, and restore test explicitly. Raw send retains
the source encryption keys, so independently preserve the recovery credential
[36].

Run a monthly scrub and monitor errors:

```sh
sudo zpool scrub <pool>
sudo zpool status -xv <pool>
```

`autotrim` defaults off. For an SSD pool, choose either tested `autotrim=on` or
a periodic `zpool trim`; upstream warns that continuous trimming can stress
some devices [35]. Snapshots, scrubs, mirrors, and RAIDZ are not backups.
