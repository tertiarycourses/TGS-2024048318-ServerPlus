# Lab 29 — Troubleshooting Common Hardware Failures

Maps to CompTIA Server+ objective **4.2 — Given a scenario, troubleshoot common hardware failures** (predictive failures; memory errors — BSOD/PSOD/dump; utilisation; POST errors; lockups; kernel panic; CMOS battery failure; system lockups; random crashes; fault/device indication — visual LED/LCD/auditory/olfactory; POST codes; misallocated virtual resources; causes: PSU fault, fans, heat sink, seated cards, incompatibility, cooling failures, backplane failure, firmware incompatibility, CPU/GPU overheating; environmental — dust, humidity, temperature; tools — event logs, firmware upgrades/downgrades, hardware diagnostics, compressed air, ESD equipment, reseating cables).

Run all commands on https://killercoda.com/playgrounds/scenario/ubuntu

**Required software (free, apt):** `dmidecode`, `lshw`, `smartmontools`, `lm-sensors`, `memtester`, `stress-ng`, `edac-utils`

```bash
apt update && apt install -y dmidecode lshw smartmontools lm-sensors memtester stress-ng edac-utils
```

---

## Step 1 — Event logs first (the rule the exam expects)

```bash
dmesg -T | tail -40
journalctl -p err -b 0 --no-pager | tail -40
journalctl -k | grep -iE 'mce|edac|temperature|throttl|over.?heat|ecc' | tail
```

Hardware faults almost always print to **dmesg / journal** before the service notices.

## Step 2 — CPU thermal + utilisation

```bash
sensors 2>/dev/null | head    # may be empty in a VM
stress-ng --cpu $(nproc) --timeout 10s --metrics-brief 2>&1 | tail
mpstat 1 3
```

Server+ 4.2 cause: **CPU/GPU overheating**. Symptoms: thermal throttling, random lockups, kernel panic. Field response: check **fans**, **heat sink seating**, dust accumulation; **compressed air**, **reseat cables**.

## Step 3 — Memory: errors, ECC, dumps

```bash
edac-util -v 2>/dev/null || echo "no EDAC kernel module exposed in this VM"
dmesg | grep -i mce | tail
journalctl -k | grep -i "memory" | tail
# memtester targets specific memory ranges — example syntax:
echo "memtester 100M 1   # would test 100 MiB, 1 pass"
```

Real hosts surface **uncorrectable ECC errors** in EDAC and in the BMC SEL log (`ipmitool sel list`). Two corrected errors on the same DIMM = swap that DIMM.

Crash dump terms the exam tests:
- **BSOD** (Blue Screen — Windows)
- **PSOD** (Purple Screen — VMware ESXi)
- **Kernel panic** (Linux)
- **Memory dump** = `/var/crash/` (Linux, via kdump) or `MEMORY.DMP` (Windows)

## Step 4 — Storage health (predictive failures via SMART)

```bash
smartctl -a /dev/sda 2>/dev/null | head -40
smartctl -H /dev/sda 2>/dev/null
smartctl -l error /dev/sda 2>/dev/null | head -20
```

SMART attributes that **predict** drive failure:
- `Reallocated_Sector_Ct`
- `Current_Pending_Sector`
- `Offline_Uncorrectable`
- `Wear_Leveling_Count` (SSD)
- `Media_Wearout_Indicator` (SSD)

When any climbs above 0 → **schedule replacement** before it fails.

## Step 5 — PSU and fan faults via IPMI

```bash
# On real hardware:
# ipmitool sensor list | grep -Ei "fan|psu|temp|voltage"
# ipmitool sel list  | head
echo "BMC SEL captures PSU input loss, fan stall, temp threshold crossings"
```

Symptoms of PSU fault: random power-off, system instability, kernel `power_meter` warnings. Pre-swap checklist (Lab 5): confirm partner PSU is healthy *first*.

## Step 6 — CMOS battery failure

Server+ 4.2 calls this out. Symptoms:
- Clock skew on every boot (`date` jumps wrong) — Lab 31 callback.
- BIOS settings reset to defaults.
- Boot order forgets the OS disk.

```bash
date
chronyc tracking 2>/dev/null | head
```

Fix: replace CR2032 battery; re-set BIOS settings; verify NTP sync.

## Step 7 — POST errors and visual / auditory cues

Server+ explicitly tests **POST codes, LED, LCD, auditory, olfactory** (yes — burnt smell is a clue).

| Cue            | Likely cause                                |
|----------------|----------------------------------------------|
| Long beep, repeating | Memory error                            |
| 3 short beeps  | Video card / GPU                             |
| No POST + amber LED | PSU / power good signal                |
| LCD: CPU thermal trip | CPU fan failure / heat sink loose     |
| Burnt smell    | Capacitor / VRM failure → power down immediately |
| Clicking HDD   | Head crash imminent → replace                |

## Step 8 — Cards, cables, and "reseating"

```bash
lspci -tv | head
lshw -short -class storage -class network 2>/dev/null | head
```

Field fix list for intermittent device drop-outs:

1. Power off; ESD strap.
2. Remove and **reseat** PCIe card and any SAS / SATA cables.
3. Inspect for bent pins, dust, corrosion.
4. **Compressed air** at the slot; don't use a vacuum (static).
5. Power on, re-check `lspci` / `dmesg`.

## Step 9 — Environmental causes

| Factor      | Effect on hardware                          | Mitigation                |
|-------------|----------------------------------------------|---------------------------|
| Dust        | Insulates components → over-temp             | Schedule cleaning         |
| Humidity high | Condensation, corrosion                    | HVAC dehumidify (Lab 19)  |
| Humidity low  | Static discharge                           | HVAC humidify, ESD wrist straps |
| Temperature | Thermal throttle, drive wear, capacitor degradation | ASHRAE 18–27 °C    |

## Step 10 — Misallocated virtual resources (VM tie-in)

For a VM that's "slow" with no obvious hardware fault:

```bash
top -bn1 | head
free -h
iostat -x 1 3
```

Common causes:
- vCPU over-subscribed (Lab 15) — host CPU at 100 %.
- Memory ballooning / swap — guest swapping is the tell.
- Storage over-subscribed — high `await`.

Resolution: move the VM or rebalance the host.

## Step 11 — Toolkit you'll carry to the rack

| Tool                | Purpose                              |
|---------------------|---------------------------------------|
| ESD wrist strap     | Prevent component damage              |
| Compressed air      | Dust removal                          |
| Multimeter          | PSU voltage check                     |
| Spare DIMMs / PSUs / drives | Hot-spare swap                |
| USB-serial cable    | Console access when LAN dies          |
| Live USB diag image | `memtest86+`, vendor diagnostic ISO   |

## Step 12 — Decision tree

```text
Symptom → check event logs first
        → no clue → check sensors (temp, fan, voltage)
            → over-temp → check fans, heat sink, dust
            → voltage fault → check PSU
        → memory log entries → run memtest86+
        → drive symptoms → smartctl -H ; replace if predictive failure
        → still unclear → reseat cards/cables → vendor diagnostic ISO
```

---

## Free online tools
- **smartmontools** — https://www.smartmontools.org/
- **memtest86+** — https://www.memtest.org/
- **EDAC docs** — https://www.kernel.org/doc/html/latest/driver-api/edac.html

## What you learned
- Event-log-first investigation discipline.
- Mapping each Server+ 4.2 symptom (BSOD/PSOD/dump, kernel panic, lockups, POST codes, CMOS battery, predictive failures, cooling) to a concrete tool.
- SMART attributes that predict drive failure.
- BMC sensor / SEL workflow for PSU and fan faults.
- Environmental causes and ESD discipline at the rack.
- Distinguishing host-hardware faults from misallocated-VM symptoms.
