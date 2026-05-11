# Free Tools Inventory — Server+ Labs

All tools below are **100% free** (open source, freeware, or free tier without a time limit). Every CLI tool is available on the disposable **Killercoda Ubuntu Playground** (https://killercoda.com/playgrounds/scenario/ubuntu) via `apt install`.

---

## Hardware inspection

| Tool | Install | Used in |
|------|---------|---------|
| `dmidecode` | `apt install dmidecode` | 1, 5, 29 |
| `lshw` | `apt install lshw` | 1, 29 |
| `lscpu` / `lsmem` / `lsblk` / `lspci` / `lsusb` | built-in (`util-linux`, `pciutils`, `usbutils`) | 1, 7, 29 |
| `smartmontools` (`smartctl`) | `apt install smartmontools` | 19, 29, 30 |
| `lm-sensors` | `apt install lm-sensors` | 19, 29 |
| `memtester` | `apt install memtester` | 29 |
| `ipmitool` | `apt install ipmitool` | 5 |
| `fwupd` / `efibootmgr` | `apt install fwupd efibootmgr` | 5 |

## Storage

| Tool | Install | Used in |
|------|---------|---------|
| `mdadm` | `apt install mdadm` | 2, 30 |
| `lvm2` | `apt install lvm2` | 3, 7 |
| `parted`, `gdisk` | `apt install parted gdisk` | 7 |
| `e2fsprogs`, `xfsprogs`, `zfsutils-linux` | `apt install e2fsprogs xfsprogs zfsutils-linux` | 7 |
| `nfs-kernel-server` | `apt install nfs-kernel-server` | 4 |
| `tgt`, `open-iscsi` | `apt install tgt open-iscsi` | 4 |
| `cryptsetup` | `apt install cryptsetup` | 18 |

## Networking

| Tool | Install | Used in |
|------|---------|---------|
| `iproute2` (`ip`, `ss`) | built-in | 8, 14, 32 |
| `bind9`, `dnsutils` (`dig`, `nslookup`) | `apt install bind9 dnsutils` | 8, 32 |
| `isc-dhcp-server` | `apt install isc-dhcp-server` | 8 |
| `ufw`, `nftables` | `apt install ufw nftables` | 9 |
| `tcpdump`, `mtr-tiny`, `traceroute` | `apt install tcpdump mtr-tiny traceroute` | 32 |
| `nmap` | `apt install nmap` | 33 |
| `haproxy`, `keepalived` | `apt install haproxy keepalived` | 14, 27 |

## Services / server roles

| Tool | Install | Used in |
|------|---------|---------|
| `nginx` | `apt install nginx` | 10, 14 |
| `samba` | `apt install samba` | 10 |
| `mariadb-server` | `apt install mariadb-server` | 10 |
| `slapd`, `ldap-utils` | `apt install slapd ldap-utils` | 11 |
| `qemu-kvm`, `libvirt-daemon-system`, `virtinst` | `apt install qemu-kvm libvirt-daemon-system virtinst` | 15 |

## Security

| Tool | Install | Used in |
|------|---------|---------|
| `openssl` | built-in | 18 |
| `openssh-server` | `apt install openssh-server` | 18 |
| `libpam-google-authenticator`, `qrencode` | `apt install libpam-google-authenticator qrencode` | 21 |
| `auditd` | `apt install auditd` | 22 |
| `rsyslog`, `logwatch` | `apt install rsyslog logwatch` | 22 |
| `lynis`, `apparmor-utils`, `debsums` | `apt install lynis apparmor-utils debsums` | 23 |
| `unattended-upgrades`, `needrestart` | `apt install unattended-upgrades needrestart` | 24 |
| `wipe`, `nwipe` | `apt install wipe nwipe` | 25 |
| `aide`, `chkrootkit` | `apt install aide chkrootkit` | 33 |

## Backup, monitoring, scripting

| Tool | Install | Used in |
|------|---------|---------|
| `rsync`, `tar`, `lftp` | `apt install rsync lftp` | 13, 26, 27 |
| `restic` | `apt install restic` | 26 |
| `lsyncd` | `apt install lsyncd` | 27 |
| `sysstat` (`sar`, `iostat`), `htop`, `iotop` | `apt install sysstat htop iotop` | 12, 30 |
| `prometheus-node-exporter` | `apt install prometheus-node-exporter` | 12 |
| `cloud-init`, `debootstrap` | `apt install cloud-init debootstrap` | 6 |
| `glpi-agent` | `apt install glpi-agent` | 17 |
| `strace`, `lsof` | `apt install strace lsof` | 30, 31 |

## Online (free) references

- **Killercoda playground** — https://killercoda.com/playgrounds/scenario/ubuntu
- **CompTIA Server+ exam objectives** — https://www.comptia.org/certifications/server
- **IP Calculator** — https://alfredang.github.io/ipcalculator/
- **SSL Labs server test** — https://www.ssllabs.com/ssltest/
- **NVD / CVE search** — https://nvd.nist.gov/
- **NIST 800-88 media sanitization** — https://csrc.nist.gov/publications/detail/sp/800-88/rev-1/final
- **NIST 800-34 contingency planning** — https://csrc.nist.gov/publications/detail/sp/800-34/rev-1/final
