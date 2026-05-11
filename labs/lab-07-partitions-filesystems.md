# Lab 7 — Partition and Volume Types, File Systems

Maps to CompTIA Server+ objective **2.1** — Partition and volume types (GPT vs. MBR, dynamic disk, LVM) and File system types (ext4, NTFS, VMFS, ReFS, ZFS).

Run all commands on https://killercoda.com/playgrounds/scenario/ubuntu

**Required software (free, apt):** `parted`, `gdisk`, `e2fsprogs`, `xfsprogs`, `zfsutils-linux`, `ntfs-3g`

```bash
apt update && apt install -y parted gdisk e2fsprogs xfsprogs zfsutils-linux ntfs-3g
```

---

## Step 1 — Create a "disk" you can partition

```bash
truncate -s 4G /srv/disk.img
losetup -P /dev/loop10 /srv/disk.img
```

## Step 2 — MBR (Master Boot Record) partition table

```bash
parted -s /dev/loop10 mklabel msdos
parted -s /dev/loop10 mkpart primary ext4 1MiB 500MiB
parted -s /dev/loop10 mkpart primary ext4 500MiB 1000MiB
parted /dev/loop10 print
```

MBR limits: max **2 TiB** disk, max **4 primary** partitions (or 3 + 1 extended). Legacy BIOS booting.

## Step 3 — Switch to GPT (GUID Partition Table)

```bash
parted -s /dev/loop10 mklabel gpt
parted -s /dev/loop10 mkpart data1 ext4 1MiB 1GiB
parted -s /dev/loop10 mkpart data2 xfs  1GiB 2GiB
parted -s /dev/loop10 mkpart data3 ext4 2GiB 3GiB
parted /dev/loop10 print
gdisk -l /dev/loop10
```

GPT: max **9.4 ZB** disk, **128 partitions** by default, redundant header (backup at end of disk), required by UEFI booting.

## Step 4 — Format with each file system

```bash
mkfs.ext4 /dev/loop10p1     # ext4   — Linux default
mkfs.xfs  /dev/loop10p2     # XFS    — large file workloads
zpool create -f tank /dev/loop10p3   # ZFS  — pooled, checksumming, snapshots
zfs list
```

## Step 5 — File system comparison (Server+ exam mapping)

| FS       | Native OS         | Use case                                              | Notes                              |
|----------|-------------------|-------------------------------------------------------|------------------------------------|
| **ext4** | Linux             | General-purpose Linux servers                         | Journaled, mature, simple          |
| **XFS**  | Linux (RHEL default) | Large files, parallel I/O, big mounts             | Online grow only (no shrink)       |
| **NTFS** | Windows           | Windows Server volumes, NTFS ACLs                     | Read/write on Linux via `ntfs-3g`  |
| **ReFS** | Windows Server    | Storage Spaces, large data integrity workloads        | Checksumming, no boot              |
| **VMFS** | VMware ESXi       | Datastore for `.vmdk` files on ESXi hosts             | Clustered, multi-host concurrent access |
| **ZFS**  | Linux/FreeBSD/Solaris | Pooled storage with checksums, snapshots, send/recv | Volume manager + FS in one         |

## Step 6 — Dynamic disk vs. LVM

`dynamic disk` is the Windows concept (spanned/striped/mirrored volumes in Disk Management). The Linux equivalent is **LVM** (Lab 3). Both let you grow, span, and mirror across multiple physical disks without changing the partition table.

```bash
pvs && vgs && lvs   # see Lab 3 — LVM is the Server+ answer on Linux
```

## Step 7 — Mounting, persistence with /etc/fstab, and UUIDs

```bash
mkdir -p /mnt/p1 /mnt/p2
mount /dev/loop10p1 /mnt/p1
blkid /dev/loop10p1
# Persist by UUID (not /dev/loopN which changes between reboots):
UUID=$(blkid -s UUID -o value /dev/loop10p1)
echo "UUID=$UUID  /mnt/p1  ext4  defaults,noatime  0 2" | tee -a /etc/fstab
mount -a
```

## Step 8 — Convert MBR → GPT non-destructively

```bash
losetup -d /dev/loop10 && losetup -P /dev/loop10 /srv/disk.img
parted -s /dev/loop10 mklabel msdos
parted -s /dev/loop10 mkpart primary ext4 1MiB 1GiB
gdisk /dev/loop10 <<'EOF'
r
g
w
y
EOF
gdisk -l /dev/loop10 | head
```

The `gdisk` recovery menu can convert MBR → GPT in place (after a full backup!).

---

## Free online tools
- **GNU parted manual** — https://www.gnu.org/software/parted/manual/
- **ZFS on Linux** — https://openzfs.github.io/openzfs-docs/
- **NTFS-3G** — https://www.tuxera.com/community/open-source-ntfs-3g/

## What you learned
- GPT vs. MBR limits and when to choose each.
- The five Server+ filesystems (ext4, NTFS, VMFS, ReFS, ZFS) and where each one lives.
- LVM as the Linux answer to Windows dynamic disks.
- UUID-based `/etc/fstab` for survivable mounts.
- Non-destructive MBR → GPT migration with `gdisk`.
