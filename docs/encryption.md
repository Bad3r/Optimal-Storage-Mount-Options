# Encryption architecture

[Project home](../README.md) | [Documentation index](README.md) | [References](references.md)

## Threat model and selected control

| Threat | Selected control | What remains exposed |
| --- | --- | --- |
| Stolen powered-off Linux disk | LUKS2 full-volume encryption | Partition layout, LUKS header, and allowed discard pattern |
| Stolen portable disk used on three desktop operating systems | VeraCrypt standard volume | Outer container size and, depending on trim behavior, allocation activity |
| Untrusted ZFS replication target | Native ZFS encryption and `zfs send -w` | Pool, dataset, snapshot hierarchy, properties, sizes, and timing |
| Independent per-user or per-directory keys | fscrypt on Ext4 or F2FS, optionally inside LUKS2 | Sizes, permissions, timestamps, extended attributes, and other metadata |
| Offline modification of a writable encrypted disk | No default control in ordinary dm-crypt | Use dm-integrity with an authenticated dm-crypt design only after accepting its complexity and cost |
| Immutable root-image tampering | Secure Boot with a signed boot chain, plus dm-verity | Writable state must be protected separately |
| Malware or root while the volume is unlocked | Storage encryption does not solve it | Application isolation, host hardening, and incident response are required |

LUKS2 protects confidentiality and provides managed keyslots, redundant
metadata, and token support. Its normal length-preserving dm-crypt modes do not
cryptographically authenticate all writable data [7], [12]. `dm-integrity` can
add per-sector integrity tags and journaling, but cryptsetup labels LUKS2
authenticated disk encryption using `--integrity` experimental and supports
only limited authenticated modes. It is excluded from the standard production
profiles unless the exact kernel, cryptsetup release, performance, failure
behavior, and recovery process have been validated [7], [14].

`dm-verity` verifies a read-only block image against a root digest [15]. The
digest or signed verity metadata must itself be authenticated by the Secure
Boot chain, such as by embedding it in a signed unified kernel image. A mutable
file on the EFI System Partition or an unsigned kernel command line is not a
trust anchor [16], [54].

Secure Boot verifies signed EFI components in the boot path; it does not
encrypt storage or authenticate normal writable user data [16]. Root encryption
also leaves the EFI System Partition visible, so a high-assurance workstation
needs a deliberately maintained Secure Boot and signed kernel or unified-kernel
image path in addition to LUKS2.

## Layer order

Use these exact architectural orders:

- **One Linux disk:** device, GPT, LUKS2, optional LVM, filesystem.
- **Conventional multi-disk RAID:** members, md RAID, LUKS2, optional LVM,
  Ext4 or XFS.
- **Btrfs native RAID:** LUKS2 on every member, then one Btrfs multi-device
  filesystem. Use the same unlock policy for every member.
- **OpenZFS:** direct whole-disk or partition vdevs, then native encrypted
  datasets. Add LUKS below each vdev only when hiding ZFS pool and dataset
  metadata is more important than operational simplicity and raw encrypted ZFS
  replication.

Do not stack LUKS2 and native ZFS encryption by default. Independently managed
keys do create two cryptographic layers, but usually add little protection for
ordinary powered-off file-data confidentiality while creating two key and
recovery systems. Use both only when LUKS must hide ZFS-visible pool and dataset
metadata and native ZFS keys or raw encrypted replication are also required.

Do not replace LUKS2 with an SSD product page that merely claims AES-256 or
self-encrypting-drive support. Hardware encryption is acceptable only after the
exact TCG Opal implementation, pre-boot unlock path, suspend behavior, firmware
update process, recovery credential, and erase semantics have been validated as
one system. This guide keeps host-controlled LUKS2 as the default.

## LUKS2 baseline

> [!CAUTION]
> `cryptsetup luksFormat` and every `mkfs` command destroy access to existing
> data on the selected target. Replace all placeholders, resolve the exact
> persistent device path, inspect it with `lsblk`, and verify the backup before
> running a destructive command.

Create and inspect a new mapping without pinning algorithms, sector size, or
PBKDF parameters to aging cargo-cult values:

```sh
sudo cryptsetup luksFormat --type luks2 /dev/disk/by-id/<exact-device>-part2
sudo cryptsetup open /dev/disk/by-id/<exact-device>-part2 cryptdata
sudo cryptsetup status cryptdata
sudo cryptsetup luksDump /dev/disk/by-id/<exact-device>-part2
```

Current cryptsetup chooses defaults from the build and device topology. Record
the actual UUID, cipher, sector size, PBKDF, data offset, and keyslots from
`luksDump`; do not force a 4096-byte encryption sector from a Gen5 label.
Cryptsetup warns that an encryption sector larger than the physical sector can
be partially written during power loss [7].

Maintain at least two unlock paths: a normal passphrase or enrolled hardware
token, and a separately stored recovery credential. Add and test a replacement
before removing the old key:

```sh
sudo cryptsetup luksAddKey /dev/disk/by-id/<exact-device>-part2
sudo cryptsetup open --test-passphrase /dev/disk/by-id/<exact-device>-part2
sudo cryptsetup luksDump /dev/disk/by-id/<exact-device>-part2
```

TPM2, FIDO2, and PKCS#11 enrollment can improve daily use, but a token or TPM
must not be the only recovery path. PCR policy is an explicit measured-boot
decision. With current systemd, omitting `--tpm2-pcrs=` means no PCR binding at
all. Use a deliberately maintained signed PCR policy or `systemd-pcrlock`
policy when measured-boot authorization is intended [11]. Test the independent
recovery key from rescue media after boot-chain changes.

Back up the LUKS header after creation and after every keyslot, token, or
metadata change:

```sh
sudo cryptsetup luksHeaderBackup \
  /dev/disk/by-id/<exact-device>-part2 \
  --header-backup-file /<offline-secure-path>/cryptdata-luks2-header.img
```

The header backup is sensitive. Anyone with the ciphertext, an old header
backup, and a credential valid in that backup can unlock the volume, including
a key that was later removed. Protect or retire obsolete header backups. A
header backup preserves encryption metadata, not user data [8].

## Discard through LUKS

dm-crypt blocks discard by default. Choose one of these policies [9]:

- **Balanced default for ordinary workstations and servers:** permit discard
  through LUKS and use the filesystem policy in
  [TRIM and discard policy](mount-options-and-trim.md#trim-and-discard-policy).
  This reveals unused regions and allocation changes, but not plaintext.
- **Strict allocation privacy:** do not permit discard. This hides allocation
  state better but can reduce space reclamation and sustained SSD performance.

Authenticated or integrity-protected LUKS2 mappings cannot pass discard through
the dm-integrity authentication-tag space. Omit `discard` for those mappings
[9].

For the balanced policy, a typical `/etc/crypttab` entry is:

```text
cryptdata UUID=<LUKS-UUID> none luks,discard
```

For strict allocation privacy, omit `discard`:

```text
cryptdata UUID=<LUKS-UUID> none luks
```

Obtain the identifier with:

```sh
sudo cryptsetup luksUUID /dev/disk/by-id/<exact-device>-part2
```

TRIM is not secure erasure. It communicates logical deallocation and gives no
universal guarantee about physical NAND remnants.

## Selective encryption

fscrypt is available on Ext4, F2FS, UBIFS, and CephFS, not released Btrfs or
XFS. It encrypts regular-file contents, filenames, and symlink targets under an
encrypted directory policy. It does not hide sizes, permissions, timestamps,
extended attributes, or sparse-hole locations, and it normally applies to an
empty directory rather than encrypting existing files in place [13].

Use fscrypt only when independently lockable user or directory trees are a
requirement. It can sit inside LUKS2 so the whole volume remains protected when
powered off. It is not a replacement for full-volume encryption and does not
protect plaintext copied to unencrypted swap. The filesystem must be created
with its `encrypt` superblock feature before an fscrypt policy can be applied;
use the conditional Ext4 or F2FS creation recipe in the
[filesystem profiles](filesystems/README.md) [13].

## Swap and hibernation

Swap can contain keys, passwords, browser state, and decrypted documents.
Select one design:

- **No hibernation:** use a swap file or logical volume inside the unlocked
  LUKS2 system, or a separate dm-crypt mapping with a fresh random key at each
  boot. The random-key crypttab design must include the `swap` option, or an
  equivalent boot-time `mkswap`, because each new key makes the previous swap
  signature unreadable. It can never support hibernation [10].
- **Hibernation:** use persistent encryption whose key can be recovered by the
  initramfs before resume. Configure the distribution's mapped `resume=` target
  and, for a swap file, its exact resume offset. A random per-boot swap key
  makes the hibernation image unrecoverable [10], [47].
- **Btrfs root:** prefer a dedicated encrypted swap partition or logical
  volume. Btrfs swap files are supported but impose NOCOW, no-compression,
  preallocation, single-device, snapshot, balance, and resume constraints
  [26].
- **OpenZFS root:** use a dedicated encrypted swap partition. The selected
  default excludes both a ZFS-hosted swap file and a swap zvol, particularly
  for hibernation, because OpenZFS documents unsupported files and reported
  zvol deadlocks [46], [55].

The default is no swap discard. `discard=once` trims the swap area at
activation, while `discard=pages` exposes page-level deallocation activity and
often provides no benefit. Enable either only after a measured device need and
an explicit allocation-leakage decision [53].
