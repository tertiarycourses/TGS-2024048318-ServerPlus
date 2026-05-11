# Lab 1 — Server Form Factors, Racking, and Power Planning

This lab maps to CompTIA Server+ objective **1.1 — Given a scenario, install physical hardware**. You cannot rack a physical server in a browser, but you can inventory a running system and plan a rack layout the same way a data-centre technician does.

Run all commands on the free Killercoda Ubuntu Playground:
https://killercoda.com/playgrounds/scenario/ubuntu

**Required software (free, installed via apt):**
- `dmidecode`, `lshw`, `pciutils`, `usbutils`

```bash
apt update && apt install -y dmidecode lshw pciutils usbutils
```

---

## Step 1 — Identify the running system as if it were a rack-mount server

```bash
dmidecode -t system | grep -E "Manufacturer|Product|Serial"
dmidecode -t chassis | grep -E "Type|Height|Power"
dmidecode -t baseboard
```

`Chassis Type` will report values such as `Rack Mount Chassis`, `Tower`, `Blade Enclosure`, or (on Killercoda) `Other` because it is a VM. On real iron the `Height` field gives you the **U** count (1U / 2U / 3U / 4U) — the unit you would use to plan rack space.

## Step 2 — Inventory CPU, RAM, bus types, and expansion cards

```bash
lscpu | head -20
lsmem || cat /proc/meminfo | head
lspci | head
lsusb
lshw -short 2>/dev/null | head -40
```

Map every line to the Server+ component list:
- CPU → `lscpu`
- Memory → `lsmem` / `/proc/meminfo`
- Bus types (PCI, PCIe) and expansion cards → `lspci`
- Interface types (SATA, SAS, USB) → `lshw -class storage`, `lsusb`

## Step 3 — Hardware compatibility list (HCL) check

A real install starts by checking the vendor HCL. Practice the lookup workflow:

1. Get the product / model string: `dmidecode -s system-product-name`.
2. Open the vendor HCL URL (Dell, HPE, Lenovo, Supermicro, Red Hat). For Ubuntu, use https://ubuntu.com/certified.
3. Confirm the **CPU family, RAID controller, and NIC** are listed for the OS you intend to install.

## Step 4 — Plan a 42U rack layout on paper

Draft a layout that respects cooling, balancing, and weight rules from objective 1.1. Example for a 42U rack:

| U   | Equipment             | Notes                                   |
|-----|-----------------------|-----------------------------------------|
| 42  | KVM 1U + cable arm    | Top, easy reach for crash cart          |
| 40-41 | Top-of-rack switch x2 | Redundant uplinks                      |
| 36-39 | 4U JBOD              | Heavy — but kept high for cabling? No → move down |
| 1-4 | 4U JBOD               | **Heaviest at bottom** (rack balancing) |
| 5-12 | 8x 1U compute         | Hot aisle / cold aisle alignment       |
| 13-16 | 4x 1U compute        |                                         |
| 17-18 | 2U UPS               | Below PDU                               |
| 19-20 | 2x PDU (left + right) | **Separate circuits / separate providers** |

Rules applied: heaviest at bottom, redundant power on **separate circuits**, cable arms for serviceability, hot-aisle/cold-aisle orientation, KVM accessible.

## Step 5 — Power budget sanity check

For each rack-mounted device, sum the **nameplate watts** and compare to the PDU + UPS rating. Rule of thumb: never load a circuit beyond 80 % of its breaker.

```bash
# Example: 12 servers × 450 W + 1 switch × 250 W + 1 KVM × 30 W = 5,680 W
# A 30 A @ 208 V circuit = 6,240 W → 5,680 W is 91 % → OVER budget. Split across 2 PDUs.
```

---

## Free online tools
- **Rack planning sheet (free)** — https://www.42u.com/rack-mount.htm
- **Dell PowerEdge HCL** — https://www.dell.com/support/home/en-us/product-support
- **Ubuntu Certified Hardware** — https://ubuntu.com/certified
- **APC UPS sizing calculator** — https://www.apc.com/configurator/

## What you learned
- How to read chassis, CPU, memory, bus and expansion card data with `dmidecode`, `lscpu`, `lspci`, `lshw`.
- The hardware compatibility list (HCL) check-before-buy workflow.
- Rack layout rules: U sizing, balancing, cooling, KVM placement, redundant power on separate circuits.
- A simple power-budget calculation against PDU and breaker limits.
