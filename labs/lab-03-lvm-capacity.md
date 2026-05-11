# Lab 3 — LVM Provisioning and Capacity Planning

Maps to CompTIA Server+ objective **1.2** (capacity planning) and **2.1** (Logical Volume Management) and **2.3** (storage management — formatting, provisioning, partitioning, page/swap location and size).

Run all commands on https://killercoda.com/playgrounds/scenario/ubuntu

**Required software (free, apt):** `lvm2`

```bash
apt update && apt install -y lvm2
```

---

## Step 1 — Create four loopback "disks"

```bash
mkdir -p /srv/disks && cd /srv/disks
for i in 1 2 3 4; do truncate -s 500M disk$i.img; losetup /dev/loop$i disk$i.img; done
```

## Step 2 — Physical Volumes (PV) → Volume Group (VG) → Logical Volumes (LV)

```bash
pvcreate /dev/loop1 /dev/loop2 /dev/loop3
vgcreate vg_data /dev/loop1 /dev/loop2 /dev/loop3
vgs
lvcreate -L 800M -n lv_web vg_data
lvcreate -L 400M -n lv_db  vg_data
lvs
```

## Step 3 — Format and mount

```bash
mkfs.ext4 /dev/vg_data/lv_web
mkfs.xfs  /dev/vg_data/lv_db
mkdir -p /srv/web /srv/db
mount /dev/vg_data/lv_web /srv/web
mount /dev/vg_data/lv_db  /srv/db
df -h | grep srv
```

## Step 4 — Capacity planning: extend a volume online

The DB volume is filling up. Extend it without downtime:

```bash
vgextend vg_data /dev/loop4              # add a new PV
lvextend -L +400M /dev/vg_data/lv_db     # grow the LV
xfs_growfs /srv/db                       # grow the filesystem (xfs is online-grow only)
df -h /srv/db
```

For ext4 the equivalent is `resize2fs /dev/vg_data/lv_web`.

## Step 5 — Page / swap location and size

Server+ 2.3 calls out *page/swap location and size*. Create a dedicated swap LV instead of a swap file:

```bash
lvcreate -L 200M -n lv_swap vg_data
mkswap /dev/vg_data/lv_swap
swapon /dev/vg_data/lv_swap
swapon --show
```

Rule of thumb (Red Hat guidance): swap size ≈ 1× RAM up to 2 GB; 0.5× RAM thereafter; databases often disable swap entirely.

## Step 6 — Snapshot a live volume (point-in-time backup)

```bash
echo "important data" > /srv/web/important.txt
lvcreate -L 100M -s -n lv_web_snap /dev/vg_data/lv_web
rm /srv/web/important.txt
# Restore: mount the snapshot read-only and copy back
mkdir -p /mnt/snap && mount -o ro /dev/vg_data/lv_web_snap /mnt/snap
cat /mnt/snap/important.txt
```

## Step 7 — Capacity vs. utilisation report

```bash
vgs --units g -o vg_name,vg_size,vg_free
lvs --units g -o lv_name,lv_size,data_percent
df -h
```

Capture these numbers as the **baseline** that you will trend in monitoring (Lab 12). The Server+ exam expects you to track capacity vs. utilisation, not just total size.

---

## Free online tools
- **LVM HOWTO** — https://tldp.org/HOWTO/LVM-HOWTO/
- **Red Hat LVM Administration** — https://access.redhat.com/documentation/

## What you learned
- PV → VG → LV provisioning model.
- Online growth of XFS and ext4 backed by LVM.
- Page/swap location, size guidance, and how to put swap on a dedicated LV.
- Point-in-time LVM snapshots for backup or pre-change rollback.
- The capacity-vs-utilisation reporting pattern the exam expects.
