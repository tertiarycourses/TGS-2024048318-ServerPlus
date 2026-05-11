# TGS-2024048318 — CompTIA Server+ SK0-005 Hands-On Labs

> **Course:** WSQ — CompTIA Certified Server+ Training
> **Course Code:** TGS-2024048318
> **Register here:** https://www.tertiarycourses.com.sg/wsq-comptia-server-training.html

These are the official hands-on lab exercises for the WSQ CompTIA Certified Server+ Training course delivered by [**Tertiary Infotech Academy Pte Ltd**](https://www.tertiarycourses.com.sg/).

A complete set of **33 step-by-step labs** aligned to the [CompTIA Server+ SK0-005 exam objectives](comptiaserverplus.pdf). Every lab runs on the free **Killercoda Ubuntu Playground** (https://killercoda.com/playgrounds/scenario/ubuntu) — no local install or physical hardware required.

---

## How to use

1. Open the Killercoda playground in your browser: https://killercoda.com/playgrounds/scenario/ubuntu
2. Pick a lab from the list below and follow the steps in order.
3. Reset the playground between labs that change firewall, storage, RAID, or service state.
4. See [labs/tools.md](labs/tools.md) for every free tool used (with install commands and download links).

> **Note on hardware labs.** Server+ is a hardware-heavy certification. Where the objective is about physical kit (racking, KVM, PSUs, hot-swap bays, BIOS/UEFI screens), the labs use the closest software equivalents available on Linux — `dmidecode`, `lshw`, `ipmitool`, `smartctl`, virtual disks via loopback files, virtual NICs via `ip link`, and KVM/QEMU virtual machines — so every concept can be reproduced inside the Killercoda VM.

---

## Lab catalogue

### Domain 1 — Server Hardware Installation and Management (18%)
- [Lab 1 — Server Form Factors, Racking, and Power Planning](labs/lab-01-hardware-racking-power.md)
- [Lab 2 — Storage Deployment with mdadm (RAID 0/1/5/6/10)](labs/lab-02-raid-mdadm.md)
- [Lab 3 — LVM Provisioning and Capacity Planning](labs/lab-03-lvm-capacity.md)
- [Lab 4 — Shared Storage with NFS and iSCSI](labs/lab-04-shared-storage-nfs-iscsi.md)
- [Lab 5 — Out-of-Band Management, BIOS/UEFI, and Firmware](labs/lab-05-oob-bios-firmware.md)

### Domain 2 — Server Administration (30%)
- [Lab 6 — Installing a Linux Server (Unattended/Scripted)](labs/lab-06-os-install-unattended.md)
- [Lab 7 — Partitions, Volumes, and File Systems](labs/lab-07-partitions-filesystems.md)
- [Lab 8 — IP, DNS, DHCP, and FQDN Configuration](labs/lab-08-ip-dns-dhcp.md)
- [Lab 9 — Firewall and Port Management](labs/lab-09-firewall-ports.md)
- [Lab 10 — Server Roles: Web, File, and Database](labs/lab-10-server-roles.md)
- [Lab 11 — Directory Services with OpenLDAP](labs/lab-11-directory-services-ldap.md)
- [Lab 12 — Performance Monitoring and Baselining](labs/lab-12-monitoring-baselining.md)
- [Lab 13 — Data Migration and Transfer (rsync, scp, SFTP)](labs/lab-13-data-transfer.md)
- [Lab 14 — High Availability: Load Balancing and NIC Teaming](labs/lab-14-high-availability.md)
- [Lab 15 — Virtualization with KVM/QEMU](labs/lab-15-virtualization-kvm.md)
- [Lab 16 — Bash Scripting for Server Administration](labs/lab-16-bash-scripting.md)
- [Lab 17 — Asset Management, Documentation, and Licensing](labs/lab-17-asset-documentation.md)

### Domain 3 — Security and Disaster Recovery (24%)
- [Lab 18 — Data at Rest (LUKS) and In Transit (TLS/SSH)](labs/lab-18-encryption-rest-transit.md)
- [Lab 19 — Physical and Environmental Security Walk-Through](labs/lab-19-physical-environmental.md)
- [Lab 20 — Identity & Access: Users, Groups, sudo, RBAC](labs/lab-20-iam-rbac-sudo.md)
- [Lab 21 — Multi-Factor Authentication with TOTP](labs/lab-21-mfa-totp.md)
- [Lab 22 — Auditing, Logging, and SIEM Basics](labs/lab-22-auditing-siem.md)
- [Lab 23 — OS and Application Hardening](labs/lab-23-os-hardening.md)
- [Lab 24 — Patch and Update Management](labs/lab-24-patching.md)
- [Lab 25 — Secure Decommissioning and Media Destruction](labs/lab-25-decommissioning-wipe.md)
- [Lab 26 — Backup Strategy: Full, Incremental, Differential, Snapshots](labs/lab-26-backup-strategy.md)
- [Lab 27 — Disaster Recovery: Replication and Site Failover](labs/lab-27-dr-replication.md)

### Domain 4 — Troubleshooting (28%)
- [Lab 28 — Troubleshooting Methodology Walk-Through](labs/lab-28-troubleshooting-methodology.md)
- [Lab 29 — Hardware Troubleshooting (dmesg, smartctl, sensors)](labs/lab-29-hardware-troubleshooting.md)
- [Lab 30 — Storage Troubleshooting (fsck, badblocks, mdadm)](labs/lab-30-storage-troubleshooting.md)
- [Lab 31 — OS and Software Troubleshooting (systemd, logs)](labs/lab-31-os-software-troubleshooting.md)
- [Lab 32 — Network Connectivity Troubleshooting](labs/lab-32-network-troubleshooting.md)
- [Lab 33 — Security Troubleshooting (ports, processes, integrity)](labs/lab-33-security-troubleshooting.md)

---

## Reference

- [labs/README.md](labs/README.md) — Lab index grouped by domain with software requirements
- [labs/tools.md](labs/tools.md) — Complete list of free tools (Killercoda + external)
- [comptiaserverplus.pdf](comptiaserverplus.pdf) — Official exam blueprint

---

## Free tools used

All tooling is **100% free** (open source, freeware, or free tier with no time limit). The bulk runs inside the disposable Killercoda VM via `apt` or a single binary download. A few labs also use free **online** services:

- **Killercoda Ubuntu Playground** — disposable Ubuntu VM in the browser (every lab) — https://killercoda.com/playgrounds/scenario/ubuntu
- **CompTIA Server+ Objectives** — official blueprint — https://www.comptia.org/certifications/server
- **IP Calculator** — subnet / IP planning (Labs 8, 32) — https://alfredang.github.io/ipcalculator/
- **SSL Labs** — TLS posture check (Lab 18) — https://www.ssllabs.com/ssltest/
- **CVE / NVD** — vulnerability lookup (Lab 24) — https://nvd.nist.gov/

Full tool list: [labs/tools.md](labs/tools.md).

---

## Legal & ethical use

Labs that involve scanning, password testing, simulated failures, or media destruction techniques (Labs 20, 22, 25, 28, 32, 33) must be performed **only** against:

- Systems you own (the Killercoda VM, your own VMs and loopback disks).
- Targets explicitly listed in a signed authorisation / Rules of Engagement.
- Public CTF / training ranges.

Unauthorised use against third-party systems is illegal in most jurisdictions, including the **Singapore Computer Misuse Act (Cap. 50A)**.
