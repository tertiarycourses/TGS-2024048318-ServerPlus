# Lab 5 — Out-of-Band Management, BIOS/UEFI, and Firmware

Maps to CompTIA Server+ objective **1.3 — Given a scenario, perform server hardware maintenance** (out-of-band management, IP KVM, BIOS/UEFI, firmware upgrades, hot-swap hardware).

Real OOB cards (iDRAC, iLO, CIMC, IPMI BMC) sit outside the OS, but you can practise the **commands and workflow** that target them. The Killercoda VM has UEFI variables you can read and a software IPMI emulator you can drive.

Run all commands on https://killercoda.com/playgrounds/scenario/ubuntu

**Required software (free, apt):** `ipmitool`, `fwupd`, `efibootmgr`, `dmidecode`

```bash
apt update && apt install -y ipmitool fwupd efibootmgr dmidecode
```

---

## Step 1 — BIOS / UEFI inventory from the running OS

```bash
dmidecode -t bios
dmidecode -s bios-version
dmidecode -s bios-release-date
[ -d /sys/firmware/efi ] && echo "Booted in UEFI mode" || echo "Booted in legacy BIOS mode"
```

Server+ 1.3 explicitly calls out **BIOS/UEFI** as a sub-objective. The distinction matters for boot order, Secure Boot, GPT support and the OOB BMC integration.

## Step 2 — UEFI boot entries

```bash
efibootmgr -v 2>/dev/null || echo "UEFI vars not exposed in this VM — on real hardware you would see entries like 0001 PXE IPv4, 0002 Ubuntu, 0003 Windows Boot Manager"
```

Practise the workflow: list entries, change order, add a network boot entry, remove a stale one — `efibootmgr -o 0002,0001,0003`, `efibootmgr -B -b 0004`.

## Step 3 — IPMI / BMC command pattern

Real servers expose a BMC on a dedicated NIC. From any admin workstation:

```bash
ipmitool -H bmc.example.com -U admin -P 'CHANGEME' chassis status
ipmitool -H bmc.example.com -U admin -P 'CHANGEME' power on
ipmitool -H bmc.example.com -U admin -P 'CHANGEME' sol activate     # Serial Over LAN console
ipmitool -H bmc.example.com -U admin -P 'CHANGEME' sensor list      # temps, fan RPM, voltages
ipmitool -H bmc.example.com -U admin -P 'CHANGEME' sel list         # System Event Log
ipmitool -H bmc.example.com -U admin -P 'CHANGEME' mc reset cold    # cold-reset the BMC itself
```

Map each command to the Server+ sub-objectives:
- `chassis power on/off/cycle` → **Remote power on/off**
- `sol activate` → **Remote console access**
- IP KVM (Java/HTML5 redirection) → **Remote drive access**, mount ISO to install OS remotely
- `sel list` → audit trail for **Remote management**

## Step 4 — Firmware upgrade workflow with `fwupd`

```bash
fwupdmgr get-devices       # what firmware is installed
fwupdmgr refresh           # pull the latest LVFS metadata
fwupdmgr get-updates       # available upgrades
# fwupdmgr update          # apply — would prompt for reboot on real hardware
```

`fwupd` is the same tool Dell, HPE, Lenovo and Intel ship firmware through on Linux. Always follow the change-management rule: **back up the running firmware, schedule downtime, document the rollback**.

## Step 5 — Local-hardware administration paths

On a real server you have four ways in if OOB is down:

1. **Crash cart** — wheeled monitor + keyboard + mouse trolley.
2. **KVM switch** — one console for many servers in the rack.
3. **Serial console** — `screen /dev/ttyS0 115200` on a USB-serial cable.
4. **Virtual administration console** — vendor app (DRAC viewer, iLO HTML5).

## Step 6 — Hot-swap hardware checklist

Server+ 1.3 lists hot-swap **drives, cages, cards, power supplies, fans**. The pre-swap checks are the same regardless of which one:

1. Confirm the **fault LED** identifies the right slot (don't pull the working drive!).
2. Verify the RAID controller marks the disk **Failed/Offline** before pulling.
3. For PSUs and fans: confirm the **redundant unit is healthy** first.
4. Document the **serial number** of the part removed and the replacement.
5. Watch `dmesg -w` and `cat /proc/mdstat` (or the controller utility) for clean re-add.

---

## Free online tools
- **LVFS firmware catalog** — https://fwupd.org/
- **OpenIPMI / FreeIPMI docs** — https://www.gnu.org/software/freeipmi/
- **Dell iDRAC / HPE iLO / Lenovo XCC / Cisco CIMC** — vendor portals (free login)

## What you learned
- BIOS vs. UEFI inventory and `efibootmgr` boot-order workflow.
- The `ipmitool` command set that maps 1:1 to Server+ OOB sub-objectives.
- Firmware update workflow with `fwupd` and the change-management guardrails.
- Crash cart, KVM, serial, virtual console — the four local-admin paths.
- Pre-swap checklist for hot-swap drives, PSUs, fans, and cards.
