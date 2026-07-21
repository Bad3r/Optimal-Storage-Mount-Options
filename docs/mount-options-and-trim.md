# Mount options and TRIM

[Project home](../README.md) | [Documentation index](README.md) | [References](references.md)

## Mount-option policy

Use the shortest option set that expresses a real policy [17]:

| Option | Policy |
| --- | --- |
| `relatime` | Keep the kernel default. It preserves useful access-time semantics while reducing updates. |
| `noatime` | Use only after confirming that no application needs access times and a benchmark shows value. It already implies `nodiratime`. |
| `lazytime` | Optional for a measured metadata-heavy workload. It delays timestamp persistence and can lose recent timestamp changes after a crash. |
| `barrier`, `barrier=1` | Do not add. Safe ordering is already the default where the option exists. |
| `nobarrier` | Never use as a generic performance option. |
| `commit=` | Keep each filesystem's default. A longer interval expands the crash-loss window. |
| `data=ordered` | Do not add to Ext4. It is already the default. |
| `discard` | Do not add generically. Follow the filesystem and encryption policy below. |
| `nodev` | Add to data and removable mounts that must not contain device nodes. |
| `nosuid` | Add where set-user-ID and set-group-ID execution is not required. |
| `noexec` | Add to content-only removable or data mounts. It is defense in depth, not a complete execution boundary because interpreters can read scripts. |
| `nofail` | Use for nonessential removable or late-available data mounts so absence does not fail boot. |
| `x-systemd.automount` | Useful for an occasionally attached filesystem on systemd; it is not portable to non-systemd init systems. |

Changing an ordinary fstab entry does not require rebuilding every initramfs.
Regenerate the initramfs only when the distribution's early-boot root,
encryption, resume, storage-driver, or hook configuration requires it. Use the
distribution's actual tool. `update-initramfs` is not an Arch Linux command.

## TRIM and discard policy

TRIM reclaims unused logical blocks. It is neither secure erase nor a reason to
change filesystems. The guide policy is:

1. For Ext4 and XFS on SSDs, use periodic trim rather than continuous
   `discard`.
2. For Btrfs on a supporting device and current kernel, allow its automatically
   selected asynchronous discard. A global weekly timer may also trim it; that
   is redundant, not a correctness conflict.
3. For F2FS, keep its device-aware default unless workload testing deliberately
   selects `nodiscard` and periodic trim.
4. For OpenZFS, test `autotrim=on` against latency, or schedule `zpool trim`.
5. Through LUKS2, discard reaches the physical SSD only when the mapping allows
   it. Accept or reject the allocation leakage explicitly.
6. Do not trim hard disks that do not advertise discard support.

For most desktop and server SSDs, util-linux documents weekly trim as sufficient
and warns that very frequent trim can hurt performance or poor-quality devices
[18]. Enable the timer, not the one-shot service:

```sh
sudo systemctl enable --now fstrim.timer
systemctl status fstrim.timer
```

One deliberate manual verification is:

```sh
sudo fstrim --all --verbose
```

Treat `not supported` as a topology result to investigate, not an error to
silence. Check the filesystem, encryption mapping, thin-provisioning layer,
RAID layer, and physical device in order.
