# CompTIA Server+ SK0-005 — Lab Index

All labs run on the free **Killercoda Ubuntu Playground**:
**https://killercoda.com/playgrounds/scenario/ubuntu**

No installs are required on your own machine — every package is pulled with `apt` (or a single binary download) inside the throw-away VM. Reset the playground between labs that change storage, firewall, RAID, or service state.

---

## Domain 1 — Server Hardware Installation and Management (18%)

| # | Lab | Free software needed |
|---|-----|----------------------|
| 1 | [Form Factors, Racking, and Power Planning](lab-01-hardware-racking-power.md) | `dmidecode`, `lshw`, `lscpu` (apt) |
| 2 | [RAID 0/1/5/6/10 with mdadm](lab-02-raid-mdadm.md) | `mdadm` (apt), loop devices |
| 3 | [LVM Provisioning and Capacity Planning](lab-03-lvm-capacity.md) | `lvm2` (apt) |
| 4 | [Shared Storage with NFS and iSCSI](lab-04-shared-storage-nfs-iscsi.md) | `nfs-kernel-server`, `tgt`, `open-iscsi` (apt) |
| 5 | [Out-of-Band Management, BIOS/UEFI, Firmware](lab-05-oob-bios-firmware.md) | `ipmitool`, `fwupd`, `efibootmgr` (apt) |

## Domain 2 — Server Administration (30%)

| # | Lab | Free software needed |
|---|-----|----------------------|
| 6 | [OS Install (Unattended/Scripted)](lab-06-os-install-unattended.md) | `cloud-init`, `debootstrap` (apt) |
| 7 | [Partitions, Volumes, File Systems](lab-07-partitions-filesystems.md) | `parted`, `gdisk`, `e2fsprogs`, `xfsprogs`, `zfsutils-linux` (apt) |
| 8 | [IP, DNS, DHCP, FQDN](lab-08-ip-dns-dhcp.md) | `iproute2`, `bind9`, `isc-dhcp-server` (apt) |
| 9 | [Firewall and Port Management](lab-09-firewall-ports.md) | `ufw`, `nftables` (apt) |
| 10 | [Server Roles: Web, File, Database](lab-10-server-roles.md) | `nginx`, `samba`, `mariadb-server` (apt) |
| 11 | [Directory Services with OpenLDAP](lab-11-directory-services-ldap.md) | `slapd`, `ldap-utils` (apt) |
| 12 | [Monitoring and Baselining](lab-12-monitoring-baselining.md) | `sysstat`, `htop`, `iotop`, `prometheus-node-exporter` (apt) |
| 13 | [Data Migration and Transfer](lab-13-data-transfer.md) | `rsync`, `openssh-client`, `lftp` (apt) |
| 14 | [High Availability: LB + NIC Teaming](lab-14-high-availability.md) | `haproxy`, `keepalived`, `ifenslave` (apt) |
| 15 | [Virtualization with KVM/QEMU](lab-15-virtualization-kvm.md) | `qemu-kvm`, `libvirt-daemon-system`, `virtinst` (apt) |
| 16 | [Bash Scripting for Server Admin](lab-16-bash-scripting.md) | `bash`, `coreutils` (built-in) |
| 17 | [Asset Management, Docs, Licensing](lab-17-asset-documentation.md) | `glpi-agent` (apt) + spreadsheet |

## Domain 3 — Security and Disaster Recovery (24%)

| # | Lab | Free software needed |
|---|-----|----------------------|
| 18 | [Encryption at Rest (LUKS) and In Transit (TLS/SSH)](lab-18-encryption-rest-transit.md) | `cryptsetup`, `openssl`, `openssh-server` (apt) |
| 19 | [Physical and Environmental Security](lab-19-physical-environmental.md) | `lm-sensors`, `smartmontools` (apt) |
| 20 | [IAM: Users, Groups, sudo, RBAC](lab-20-iam-rbac-sudo.md) | `sudo`, `libpam-pwquality` (apt) |
| 21 | [MFA with TOTP (PAM)](lab-21-mfa-totp.md) | `libpam-google-authenticator`, `qrencode` (apt) |
| 22 | [Auditing, Logging, SIEM Basics](lab-22-auditing-siem.md) | `auditd`, `rsyslog`, `logwatch` (apt) |
| 23 | [OS and Application Hardening](lab-23-os-hardening.md) | `lynis`, `apparmor-utils`, `debsums` (apt) |
| 24 | [Patch and Update Management](lab-24-patching.md) | `unattended-upgrades`, `needrestart` (apt) |
| 25 | [Decommissioning and Media Destruction](lab-25-decommissioning-wipe.md) | `coreutils` (`shred`, `dd`), `wipe`, `nwipe` (apt) |
| 26 | [Backup Strategy](lab-26-backup-strategy.md) | `tar`, `rsync`, `restic` (apt) |
| 27 | [DR: Replication and Site Failover](lab-27-dr-replication.md) | `rsync`, `lsyncd`, `keepalived` (apt) |

## Domain 4 — Troubleshooting (28%)

| # | Lab | Free software needed |
|---|-----|----------------------|
| 28 | [Troubleshooting Methodology](lab-28-troubleshooting-methodology.md) | built-in tools |
| 29 | [Hardware Troubleshooting](lab-29-hardware-troubleshooting.md) | `dmesg`, `smartmontools`, `lm-sensors`, `memtester` (apt) |
| 30 | [Storage Troubleshooting](lab-30-storage-troubleshooting.md) | `e2fsprogs`, `mdadm`, `lsof`, `iotop` (apt) |
| 31 | [OS / Software Troubleshooting](lab-31-os-software-troubleshooting.md) | `systemd`, `journalctl`, `strace`, `dpkg` (built-in) |
| 32 | [Network Troubleshooting](lab-32-network-troubleshooting.md) | `iproute2`, `iputils-ping`, `dnsutils`, `tcpdump`, `mtr-tiny`, `traceroute` (apt) |
| 33 | [Security Troubleshooting](lab-33-security-troubleshooting.md) | `nmap`, `lsof`, `aide`, `chkrootkit` (apt) |

---

## Suggested order

Follow Domain 1 → 2 → 3 → 4 in numeric order. Each lab is self-contained. Reset the Killercoda playground between labs that change firewall, storage, or network state.

## Tools reference

See [tools.md](tools.md) for the complete free-software inventory used across all 33 labs, grouped by category and lab number.
