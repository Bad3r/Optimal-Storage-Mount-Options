# What PCIe 5.0 NVMe Changes

[Project home](../README.md) | [Documentation index](README.md) | [References](references.md)

M.2 is a form factor, PCIe is a transport, NVMe is a storage protocol, and the
filesystem is a higher software layer. PCIe 5.0 runs at 32 GT/s per lane, so a
four-lane device has roughly 16 GB/s of raw link bandwidth in each direction
before protocol overhead [1], [2]. That bandwidth does not change filesystem
crash semantics or make a mount option safer.

Linux's multi-queue block layer already maps per-CPU submission queues to
hardware dispatch queues. Completion order is not guaranteed by the block
layer, so filesystems still need ordering, flushes, and application durability
calls [3]. A Gen5 device mainly changes where bottlenecks appear:

- Encryption, checksumming, and compression can become CPU or memory-bandwidth
  limited before the SSD reaches its sequential specification.
- Peak throughput can trigger thermal throttling. Cooling requirements are
  model-specific; follow the SSD and motherboard manuals. For example, a
  primary Crucial T700 document requires a heatsink and adequate airflow for
  the non-heatsink model to avoid throttling [6].
- High queue-depth vendor figures do not predict desktop latency, database
  durability, encrypted throughput, or a nearly full filesystem.
- Power-loss protection remains a product property, not an NVMe-generation
  property.

There is no `gen5`, `nvme`, `m2`, or universal `ssd` mount option to add.
Do not reformat an NVMe namespace to 4 KiB LBAs, force a scheduler, change
read-ahead, override queue depth, or add dm-crypt workqueue flags merely because
the label says Gen5. Inspect the topology and benchmark the encrypted,
mounted filesystem with the real workload first.

## Read-only inventory

Install `nvme-cli` and `smartmontools` from the distribution, then collect a
baseline [5]:

```sh
lsblk -d -o NAME,MODEL,TRAN,ROTA,LOG-SEC,PHY-SEC,MIN-IO,OPT-IO,DISC-GRAN,DISC-MAX
lsblk -f
sudo nvme list
sudo nvme list-subsys
sudo nvme id-ctrl -H /dev/nvme0
sudo nvme id-ns -H /dev/nvme0n1
sudo nvme smart-log /dev/nvme0
sudo nvme error-log /dev/nvme0
sudo nvme self-test-log /dev/nvme0
sudo smartctl -x /dev/nvme0
```

Trend composite and individual temperatures, thermal-throttling time,
`critical_warning`, `available_spare`, `percentage_used`, unexpected power
losses, media and data-integrity errors, and firmware revision. A
`percentage_used` value of 100 is a vendor endurance estimate, not an immediate
failure declaration, and the NVMe error-log count must be interpreted from its
status entries rather than treated as a NAND-failure counter [1].

Never use `nvme format`, `nvme sanitize`, `nvme write-zeroes`, or raw
`blkdiscard` as an inspection or health command. They can destroy data.
