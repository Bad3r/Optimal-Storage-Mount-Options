# Portable and Special-Purpose Filesystems

[Project overview](../../README.md) | [Documentation](../README.md) |
[Filesystem index](README.md) | [References](../references.md)

## exFAT, FAT32, NTFS, and VeraCrypt

Base exFAT has no metadata journal, Unix ownership, POSIX permissions, links,
or native filesystem encryption [39], [40]. TexFAT is a separate transactional
extension and must not be confused with ordinary exFAT. Use current
`exfatprogs` defaults when creating exchange media [41].

For a fixed private single-user exFAT mount:

```fstab
UUID=<EXFAT-UUID>  /media/exchange  exfat  nofail,nodev,nosuid,noexec,uid=<UID>,gid=<GID>,fmask=0177,dmask=0077  0  0
```

Use actual numeric IDs, not a universal `1000`. Remove `noexec` only if
executing files from the medium is an intentional requirement. Desktop
automounters normally synthesize ownership without a fixed fstab entry. Always
unmount or safely eject the medium, and never keep the only copy on exFAT.

For confidential cross-platform media, use a current VeraCrypt standard volume
with exFAT inside. VeraCrypt supports current Linux, Windows, and macOS releases
[43]. Use its GUI or an interactive prompt so credentials do not enter shell
history. On Linux, VeraCrypt's default native kernel cryptographic path does not
block TRIM. If allocation privacy requires blocking it, disable kernel
cryptographic services in VeraCrypt preferences or use
`--mount-options=nokernelcrypto`, then measure the performance cost [44]. This
guide does not select hidden volumes. If an outer volume does contain one,
mount the outer volume read-only or enable hidden-volume protection on every
writable mount; unprotected outer writes or discard can overwrite the hidden
volume. Keep a separate encrypted backup because neither exFAT nor the
container format replaces redundancy and restore testing.

Use FAT32 only for an EFI System Partition or a device that explicitly requires
it. The Linux VFAT driver synthesizes owners and modes from mount options; the
format does not store Unix ownership or permissions [42]. Use NTFS when Windows
owns and repairs the volume, not for a new Linux root or Linux-owned application
store.

## Media of unknown provenance

This guide assumes the operator owns and trusts the hardware. Removable media
that was found, given, or bought outside a retail channel does not satisfy that
assumption, and the questions it raises are prior to filesystem selection:
whether the advertised capacity is real, whether the device advertises
interfaces beyond mass storage, and whether the controller firmware can be
trusted at all. Formatting answers none of them.

Triage procedure and the corresponding threat model are maintained separately in
[Untrusted removable media triage](https://github.com/Bad3r/usb-triage).

## tmpfs: not an SSD-wear policy

Do not create a fixed 8 GiB `/tmp` tmpfs merely to reduce SSD writes. tmpfs is
volatile, can use swap by default, and defaults to a size limit based on memory
when no size is specified [45]. Use it only when volatile semantics and bounded
memory consumption fit the workload. Secret tmpfs pages still require encrypted
swap. Distribution-managed `/tmp` behavior should normally remain unchanged.
