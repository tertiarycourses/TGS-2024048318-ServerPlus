# Lab 30 — Storage Troubleshooting

Maps to CompTIA Server+ objective **4.3 — Given a scenario, troubleshoot storage problems** (boot errors, sector block errors, cache battery failure, read/write errors, failed drives, page/swap/scratch file errors, partition errors, slow file access, OS not found, unsuccessful backup, unable to mount, drive not available, cannot access logical drive, data corruption, slow I/O, restore failure, cache failure, multiple drive failure; causes — disk space utilisation, misconfigured RAID, media failure, drive failure, controller failure, HBA failure, loose connectors, cable problems, misconfiguration, corrupt boot sector, corrupt filesystem table, array rebuild, improper partition, bad sectors, cache battery failure, cache turned off, insufficient space, improper RAID configuration, mismatched drives, backplane failure; tools — partitioning, disk management, RAID/array management, system logs, mount, `net use`, monitoring, visual/auditory inspection).

Run all commands on https://killercoda.com/playgrounds/scenario/ubuntu

**Required software (free, apt):** `mdadm`, `lvm2`, `e2fsprogs`, `xfsprogs`, `parted`, `gdisk`, `smartmontools`, `lsof`, `iotop`, `sysstat`

```bash
apt update && apt install -y mdadm lvm2 e2fsprogs xfsprogs parted gdisk smartmontools lsof iotop sysstat
```

---

## Step 1 — Capacity / disk space utilisation

```bash
df -h
df -i             # inode usage — full inodes ≠ full bytes
du -xhd 1 / 2>/dev/null | sort -h | tail
```

"Disk full" with `df` low usage = **inodes exhausted** (millions of small files). The fix is to delete or move small files, not extend the volume.

## Step 2 — Slow I/O and IOPS

```bash
iostat -xz 1 5
iotop -bo -n 3 2>/dev/null | head
```

High `%util` + high `await` → the disk is saturated. Check who's hammering it (`iotop`) and the RAID controller cache state (cache OFF causes slow writes — Server+ 4.3 calls this out).

## Step 3 — Slow file access — fragmentation, atime, journal

```bash
mount | grep -E "noatime|relatime"
tune2fs -l $(findmnt -no SOURCE /) 2>/dev/null | head -30
```

Quick wins:
- Mount with `noatime` to skip access-time updates.
- For ext4 large directories, enable `dir_index` (`tune2fs -O dir_index`).

## Step 4 — Boot errors / "OS not found" / corrupt boot sector

| Symptom                       | Likely cause                          | Fix                                    |
|-------------------------------|----------------------------------------|----------------------------------------|
| "OS not found"                | BIOS boot order changed; disk dead     | Check BIOS, swap disk                  |
| "Missing operating system"    | Corrupt MBR/GPT                        | `boot-repair`, `grub-install`          |
| "GRUB rescue"                 | /boot partition gone or filesystem trashed | Rescue ISO + `fsck` + reinstall GRUB |
| Windows: "BOOTMGR is missing" | NTFS boot sector damaged               | `bootrec /fixboot /fixmbr`             |

```bash
gdisk -l /dev/sda 2>/dev/null | head -10
```

## Step 5 — Unable to mount / drive not available

```bash
truncate -s 100M /srv/badfs.img
mkfs.ext4 /srv/badfs.img
# Corrupt the superblock
dd if=/dev/urandom of=/srv/badfs.img bs=1K count=4 conv=notrunc 2>/dev/null
mkdir -p /mnt/badfs
mount -o loop /srv/badfs.img /mnt/badfs 2>&1 | head    # mount fails
```

Recovery — use a **backup superblock**:

```bash
LOOP=$(losetup -fP --show /srv/badfs.img)
mkfs.ext4 -n $LOOP 2>&1 | tail -5            # prints backup superblock locations
fsck.ext4 -b 32768 $LOOP || true
mount $LOOP /mnt/badfs 2>&1 | head
```

The Server+ pattern: `fsck` (Linux) / `chkdsk` (Windows) → if still bad, restore from backup (Lab 26).

## Step 6 — Filesystem table & partition errors

```bash
parted -l 2>&1 | head -20
sgdisk -v /dev/sda 2>&1 | head     # verify GPT
```

Repair workflows:
- **MBR** corrupted: `testdisk` to rebuild.
- **GPT** primary header bad: `sgdisk -e /dev/sdX` rebuilds from the backup header at end of disk.
- **Filesystem table corrupt** (Windows): `chkdsk /f`.

## Step 7 — Failed drive / multiple drive failure in RAID

```bash
mkdir -p /srv/d && cd /srv/d
for i in 1 2 3 4; do truncate -s 100M d$i.img; losetup /dev/loop3$i d$i.img; done
mdadm --create /dev/md30 --level=5 --raid-devices=4 /dev/loop31 /dev/loop32 /dev/loop33 /dev/loop34
cat /proc/mdstat

# Fail one drive
mdadm /dev/md30 --fail /dev/loop32 --remove /dev/loop32
cat /proc/mdstat

# Add a spare
truncate -s 100M d5.img && losetup /dev/loop35 d5.img
mdadm /dev/md30 --add /dev/loop35
watch -n1 -d cat /proc/mdstat
```

**Multiple drive failure** in RAID 5 → array is gone. Restore from backup. Lesson: RAID 5 is fragile for large drives; RAID 6 (Lab 2) is the production answer.

## Step 8 — Cache battery failure and "cache turned off"

Server+ specifically names **cache battery failure** and **cache turned off**:

- Controllers disable write-back cache when the battery / capacitor is bad → writes become write-through → **massive write slowdown**.
- Field response: replace BBU / supercap; re-enable write-back.

On Linux you see this through the controller utility (`storcli`, `megacli`, `ssacli`). Inside the OS, watch for sudden `await` spikes that coincide with the BBU warning.

## Step 9 — Unsuccessful backup / restore failure

Trigger and diagnose:

```bash
tar -czf /srv/backup-test.tgz /etc 2>&1 | tail
echo "exit code: $?"
```

Common causes from objective 4.3:
- **Insufficient space** at destination — `df -h` the target.
- **Permissions** — backup runner can't read source.
- **Media failure** — tape error, disk read error.
- **Open files / locks** — see Lab 26 open-file backup section.

Tools to investigate:

```bash
journalctl -u restic.timer -n 50 --no-pager 2>/dev/null
lsof | grep /srv/data | head
```

## Step 10 — Cable / loose-connector / backplane fault clues

Hints from the OS:

```bash
dmesg | grep -iE "ata|scsi|sas|link rate|hard reset" | tail
journalctl -k | grep -iE "drop|enclosure|backplane" | tail
```

Repeated link-rate negotiations or `hard reset` events = bad cable, bad SFF connector, or **backplane** issue. Reseat or swap.

## Step 11 — Page / swap / scratch file errors

```bash
swapon --show
free -h
journalctl -k | grep -iE "swap|oom" | tail
```

Symptoms the exam expects: "page/swap/scratch file or partition" errors → server thrashes or OOM-kills. Fix: enlarge swap (Lab 3 step 5) or, better, give the workload more RAM.

## Step 12 — Documentation & tools mapping (objective 4.3)

| Server+ tool / technique     | Linux equivalent / use                  |
|------------------------------|------------------------------------------|
| Partitioning tools           | `parted`, `gdisk`, `fdisk`               |
| Disk management              | `lsblk`, `lvm` tools, GNOME Disks         |
| RAID and array management    | `mdadm`, vendor `storcli` / `ssacli`     |
| System logs                  | `dmesg`, `journalctl`, `/var/log/syslog` |
| Disk mounting (`net use`)    | Windows: `net use Z: \\srv\share`; Linux: `mount -t cifs ...` |
| Monitoring tools             | `iostat`, `iotop`, `sar -d`              |
| Visual / auditory            | LEDs on caddy + bay; clicking HDD        |

---

## Free online tools
- **smartmontools** — https://www.smartmontools.org/
- **testdisk + photorec** — https://www.cgsecurity.org/
- **mdadm wiki** — https://raid.wiki.kernel.org/index.php/Linux_Raid

## What you learned
- Diagnosing disk-full, inode-full, slow I/O, and slow file access.
- Recovery patterns for missing OS, corrupt superblocks, and corrupt partition tables.
- mdadm RAID failure + rebuild workflow; why RAID 6 beats RAID 5 for big drives.
- Cache battery / BBU implications for write performance.
- The OS-side clues for cable, connector, and backplane faults.
- Page / swap-related troubleshooting tied back to Lab 3.
- The full Server+ 4.3 tools-and-techniques mapping.
