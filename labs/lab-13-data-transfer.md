# Lab 13 — Data Migration and Transfer

Maps to CompTIA Server+ objective **2.3** — Data migration and transfer: infiltration, exfiltration, disparate OS data transfer, Robocopy, file transfer, fast copy, SCP.

Run all commands on https://killercoda.com/playgrounds/scenario/ubuntu

**Required software (free, apt):** `rsync`, `openssh-client`, `openssh-server`, `lftp`, `pv`

```bash
apt update && apt install -y rsync openssh-client openssh-server lftp pv
systemctl start ssh
```

---

## Step 1 — Set up source and destination trees

```bash
mkdir -p /srv/src /srv/dst
for i in $(seq 1 50); do dd if=/dev/urandom of=/srv/src/file$i.bin bs=1K count=$((RANDOM%200+10)) 2>/dev/null; done
du -sh /srv/src
```

## Step 2 — `rsync` — the workhorse for server-to-server transfer

```bash
rsync -avh /srv/src/ /srv/dst/
rsync -avh --dry-run --delete /srv/src/ /srv/dst/   # show what *would* sync
rsync -avh --partial --progress /srv/src/ /srv/dst/
```

`-a` = archive (preserves perms, times, symlinks). `--delete` makes dst an exact mirror. `--partial` resumes interrupted transfers.

Over the network:

```bash
rsync -avzh -e ssh /srv/src/  user@remote:/srv/dst/
```

## Step 3 — SCP (Secure Copy Protocol)

```bash
scp -r /srv/src/ root@127.0.0.1:/srv/dst-scp/    # accept the host key
scp -p file1.bin root@127.0.0.1:/tmp/             # preserve timestamps
```

SCP is simple but **no resume, no delta**. Use it for one-off small copies; use rsync for bulk and ongoing sync.

## Step 4 — SFTP (interactive file transfer over SSH)

```bash
sftp root@127.0.0.1 <<'EOF'
ls /srv
lcd /tmp
put /etc/hostname
quit
EOF
```

## Step 5 — Disparate-OS transfer (Linux ↔ Windows)

Server+ explicitly mentions **Robocopy** — that is the Windows equivalent of `rsync`. The cross-OS transfer choices are:

| Direction              | Recommended tool                                  |
|------------------------|---------------------------------------------------|
| Linux → Linux          | `rsync` (over SSH)                                 |
| Windows → Windows      | **Robocopy** (`robocopy src dst /MIR /Z /R:3`)    |
| Windows → Linux        | SMB mount + `rsync`, or WinSCP                    |
| Linux → Windows        | Samba share + Robocopy, or `rsync` to Cygwin/WSL |

```bash
# Linux side: mount the Windows share, then rsync into it
# mount -t cifs //win/share /mnt/win -o username=admin,password=...
# rsync -avh /srv/src/ /mnt/win/dst/
```

## Step 6 — Fast copy of very large files

```bash
# Show throughput while copying:
pv /srv/src/file1.bin > /srv/dst/file1.bin

# Direct I/O (skip the page cache) for huge migrations:
dd if=/srv/src/file1.bin of=/srv/dst/file1.bin bs=1M oflag=direct status=progress

# Multi-threaded local copy with rsync + xargs:
ls /srv/src | xargs -n1 -P8 -I{} cp /srv/src/{} /srv/dst/
```

## Step 7 — FTP / FTPS / SFTP via `lftp` (legacy compatibility)

```bash
# lftp -e "set ftp:ssl-allow yes; mirror -R /srv/src /backup; quit" ftpuser@ftp.example.com
echo "lftp script syntax shown — no public FTP server reachable from Killercoda by default"
```

## Step 8 — Infiltration vs. exfiltration awareness

Server+ 2.3 lists *infiltration* and *exfiltration* under data transfer. The same `rsync` / `scp` commands that legitimately migrate data also exfiltrate it. Controls:

1. **Egress firewall**: outbound SSH/SFTP only to whitelisted destinations (Lab 9).
2. **DLP** monitoring on file-share access (objective 3.4).
3. **Auditd watch** on sensitive directories (Lab 22).
4. **Bandwidth caps** on bulk-copy users (`rsync --bwlimit=1024`).

```bash
rsync -avh --bwlimit=1024 /srv/src/ user@remote:/srv/dst/
```

---

## Free online tools
- **rsync manual** — https://rsync.samba.org/
- **OpenSSH manual** — https://www.openssh.com/manual.html
- **Microsoft Robocopy reference** — https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/robocopy
- **WinSCP (free, Windows)** — https://winscp.net/

## What you learned
- `rsync` archive + delta + delete + bandwidth-limit options.
- When to use SCP (simple) vs. SFTP (interactive) vs. rsync (bulk).
- Cross-OS choices including the Windows **Robocopy** equivalent.
- Fast-copy techniques for very large files (`pv`, `dd oflag=direct`, parallel `xargs`).
- How the same transfer tools enable both legitimate migration **and** exfiltration, and the controls that distinguish them.
