# Lab 19 — Physical and Environmental Security Walk-Through

Maps to CompTIA Server+ objective **3.2 — Summarize physical security concepts** (physical access controls: bollards, architectural reinforcements — signal blocking, reflective glass, datacenter camouflage; fencing; security guards; security cameras; locks — biometric, RFID, card readers; mantraps; safes; environmental controls — fire suppression, HVAC, sensors).

This is a hardware concept lab. Use the Killercoda VM only to **inspect environmental sensors** the OS can see; do the rest as a written walk-through.

Run all commands on https://killercoda.com/playgrounds/scenario/ubuntu

**Required software (free, apt):** `lm-sensors`, `smartmontools`

```bash
apt update && apt install -y lm-sensors smartmontools
```

---

## Step 1 — Physical access control layers (defence in depth)

A real data centre stacks controls from outside in. Map each layer to its purpose:

| Layer (outside → in) | Control                       | Purpose                              |
|----------------------|--------------------------------|--------------------------------------|
| Property perimeter   | Fencing, bollards              | Stop vehicles, channel pedestrians   |
| Building            | Datacenter camouflage          | Avoid drawing attention              |
| Lobby               | Security guards, cameras       | Detect + deter                       |
| Inner doors         | Card reader + biometric (2-factor) | Authentication + audit            |
| Trap between zones  | Mantrap / airlock              | Prevent tailgating                   |
| Server hall         | Smart-card cabinets, safes for KMS / HSM | Last line of defence       |

Bollards = anti-ram posts. Architectural reinforcements = signal blocking (TEMPEST / Faraday), reflective glass, blast walls.

## Step 2 — Locks the exam asks about

| Lock type          | Authenticator                  | Replay-resistant?     |
|--------------------|--------------------------------|------------------------|
| Mechanical key     | Physical key                   | No — easy duplication |
| Card reader        | Magstripe / proximity card     | No — cards clonable   |
| **RFID**           | RFID badge (often + PIN)       | Partly — challenge protocols |
| **Biometric**      | Fingerprint, iris, face        | Yes — but template needs protection |

Best practice: **two-factor** — card + biometric, or card + PIN.

## Step 3 — Environmental controls

```bash
sensors-detect --auto >/dev/null 2>&1 || true
sensors 2>/dev/null | head -30 || echo "no exposed sensors in this VM — on real hardware lm-sensors lists CPU / mainboard temps"
smartctl -a /dev/sda 2>/dev/null | grep -i temperature | head -3 || true
```

Server+ 3.2 lists these environmental controls:

| Control        | Target                                              |
|----------------|------------------------------------------------------|
| Fire suppression | Inert gas (FM-200, Novec 1230) — no water on electronics |
| HVAC           | 18–27 °C, 40–60 % RH per ASHRAE TC 9.9              |
| Sensors        | Temp, humidity, smoke, water-leak, door-open        |

## Step 4 — Cameras and guards as detective controls

| Control          | Category                                  |
|------------------|--------------------------------------------|
| CCTV             | Detective (and deterrent if visible)       |
| Guard patrol     | Detective + responder                      |
| Motion sensors   | Detective                                  |
| Door alarm       | Detective                                  |
| Visitor log      | Detective + audit trail                    |

Cameras retain footage for **30–90 days** typically (link to retention policy from Lab 17).

## Step 5 — Datacenter physical zones

Draw four concentric rings:

1. **Perimeter** — fence, bollards, gatehouse, vehicle screening.
2. **Building shell** — windowless / reflective glass, signal-blocking walls.
3. **Operations area** — mantrap, NOC, break rooms, no servers.
4. **Server hall** — biometric + card, locked cabinets, hot/cold aisle (Lab 1).

## Step 6 — Tailgating, piggybacking, and the mantrap

A mantrap admits **one person at a time**. The inner door cannot open until the outer door closes and the person inside passes a second factor. This defeats tailgating (unauthorised follow-through).

## Step 7 — Sample physical-security checklist

```text
[ ] Perimeter fence intact, gate alarms tested in last 90 days
[ ] CCTV coverage of every entry, NVR retention ≥ 30 days
[ ] Visitor badges colour-coded, escorted at all times
[ ] Card-reader access list reviewed quarterly
[ ] Biometric template database encrypted at rest
[ ] HVAC redundancy N+1, set 22 °C ±2, humidity 45 % ±10
[ ] Fire suppression inspected annually
[ ] Water-leak sensors under raised floor, alerts to NOC
[ ] Mantrap interlock tested monthly
[ ] Cabinet locks with audit log
```

---

## Free online tools
- **Uptime Institute Tier standards** — https://uptimeinstitute.com/tiers
- **ASHRAE thermal guidelines** — https://www.ashrae.org/
- **NIST SP 800-116 (PIV / physical access)** — https://csrc.nist.gov/publications/sp

## What you learned
- The full Server+ 3.2 vocabulary: bollards, architectural reinforcements, mantraps, RFID, biometric, signal blocking, reflective glass, datacenter camouflage.
- Lock and authenticator pros / cons, and why two-factor physical access matters.
- ASHRAE environmental targets and the sensor set a server room needs.
- Camera retention vs. policy.
- Concentric-ring data-centre zoning.
- A reusable physical-security audit checklist.
