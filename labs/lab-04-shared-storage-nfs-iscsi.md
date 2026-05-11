# Lab 4 — Shared Storage: NAS (NFS / CIFS) and SAN (iSCSI)

Maps to CompTIA Server+ objective **1.2** — Shared storage: NAS (NFS, CIFS), SAN (iSCSI, Fibre Channel, FCoE). Fibre Channel and FCoE require dedicated HBAs and switches, so this lab focuses on the two protocols you can run end-to-end inside the Killercoda VM: **NFS** (file-level NAS) and **iSCSI** (block-level SAN).

Run all commands on https://killercoda.com/playgrounds/scenario/ubuntu

**Required software (free, apt):** `nfs-kernel-server`, `nfs-common`, `tgt`, `open-iscsi`, `samba`

```bash
apt update && apt install -y nfs-kernel-server nfs-common tgt open-iscsi samba
```

---

## Step 1 — Export an NFS share (file-level / NAS)

```bash
mkdir -p /srv/nfs/share && echo "hello from NAS" > /srv/nfs/share/readme.txt
echo "/srv/nfs/share  127.0.0.1(rw,sync,no_subtree_check,no_root_squash)" >> /etc/exports
exportfs -ra
systemctl restart nfs-kernel-server
showmount -e 127.0.0.1
```

Mount it from the same VM acting as the "client":

```bash
mkdir -p /mnt/nas && mount -t nfs 127.0.0.1:/srv/nfs/share /mnt/nas
cat /mnt/nas/readme.txt
mount | grep nfs
```

## Step 2 — CIFS / SMB share (Windows-style NAS)

```bash
mkdir -p /srv/cifs && echo "hello from CIFS" > /srv/cifs/readme.txt
cat >> /etc/samba/smb.conf <<'EOF'
[lab]
   path = /srv/cifs
   browseable = yes
   guest ok = yes
   read only = no
EOF
systemctl restart smbd
smbclient -L //127.0.0.1 -N
smbclient //127.0.0.1/lab -N -c 'ls'
```

NFS and CIFS are both **file-level**. The server keeps the filesystem; the client sees files and directories.

## Step 3 — Build an iSCSI target (block-level / SAN)

Create a 500 MB backing store and export it as a LUN:

```bash
truncate -s 500M /srv/lun0.img
cat > /etc/tgt/conf.d/lab.conf <<'EOF'
<target iqn.2026-05.com.example:lun0>
    backing-store /srv/lun0.img
    initiator-address 127.0.0.1
</target>
EOF
systemctl restart tgt
tgtadm --mode target --op show
```

## Step 4 — Connect from the iSCSI initiator

```bash
iscsiadm -m discovery -t st -p 127.0.0.1
iscsiadm -m node --login
lsblk | tail        # a new /dev/sdX (or /dev/sdN) appears — this is the LUN
```

Format and mount the new SAN LUN as if it were a local disk:

```bash
NEW=$(lsblk -ndo NAME,TYPE | awk '$2=="disk"{print $1}' | tail -1)
mkfs.ext4 /dev/$NEW
mkdir -p /mnt/san && mount /dev/$NEW /mnt/san
echo "block-level data" > /mnt/san/readme.txt
df -h /mnt/san
```

## Step 5 — Compare NAS vs. SAN vs. FC / FCoE

| Feature              | NFS / CIFS (NAS)     | iSCSI (SAN — IP)        | Fibre Channel / FCoE  |
|----------------------|-----------------------|--------------------------|------------------------|
| Access type          | File                  | Block (raw LUN)          | Block (raw LUN)        |
| Transport            | TCP/IP                | TCP/IP                   | FC (dedicated) / Ethernet |
| Typical use          | User shares, archives | Databases, VM datastores | High-throughput SANs   |
| HBA / switch needed  | NIC only              | NIC (or iSCSI HBA)       | FC HBA, FC switch / DCB |
| Cost                 | Lowest                | Low                      | Highest                |

---

## Free online tools
- **Linux NFS HOWTO** — https://nfs.sourceforge.net/nfs-howto/
- **Samba documentation** — https://www.samba.org/samba/docs/
- **tgt (Linux SCSI target)** — https://stgt.sourceforge.net/

## What you learned
- File-level (NFS / CIFS) vs. block-level (iSCSI / FC) shared storage.
- Building an iSCSI target with `tgt` and connecting an initiator with `iscsiadm`.
- Why a SAN LUN appears to the OS as a plain disk it must format itself.
- Where FC and FCoE fit relative to iSCSI for the SK0-005 exam.
