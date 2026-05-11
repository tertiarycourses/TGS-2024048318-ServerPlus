# Lab 17 — Asset Management, Documentation, and Licensing

Maps to CompTIA Server+ objectives **2.7 — Asset management and documentation** (labelling, warranty, leased vs. owned, life-cycle, inventory, serial number, asset tag, service manuals, architecture/infrastructure/workflow diagrams, recovery processes, baselines, change management, server configurations, BIA, MTBF, MTTR, RPO, RTO, SLA, uptime requirements, secure documentation storage) **and 2.8 — Licensing concepts** (per-instance, per-user, per-server, per-socket, per-core, site-based, physical vs. virtual, node-locked, signatures, open source, subscription, license vs. maintenance and support, volume licensing, true-up, version compatibility).

Run all commands on https://killercoda.com/playgrounds/scenario/ubuntu

**Required software:** built-in tools + an offline spreadsheet or CSV editor.

---

## Step 1 — Auto-discover the asset details

```bash
{
  echo "asset_tag,$(hostname -f)"
  echo "make,$(dmidecode -s system-manufacturer)"
  echo "model,$(dmidecode -s system-product-name)"
  echo "serial,$(dmidecode -s system-serial-number)"
  echo "uuid,$(dmidecode -s system-uuid)"
  echo "bios_version,$(dmidecode -s bios-version)"
  echo "cpu,$(lscpu | awk -F: '/Model name/{print $2}' | xargs)"
  echo "ram_gb,$(free -g | awk '/Mem/{print $2}')"
  echo "os,$(. /etc/os-release; echo $PRETTY_NAME)"
  echo "ip,$(hostname -I | awk '{print $1}')"
  echo "discovered_at,$(date -Is)"
} > /root/asset-$(hostname).csv
cat /root/asset-$(hostname).csv
```

This is the **inventory** + **make/model** + **serial number** + **asset tag** information from objective 2.7.

## Step 2 — Life-cycle stages

| Stage         | Activity                                            |
|---------------|-----------------------------------------------------|
| Procurement   | RFQ, vendor selection, HCL check, PO, warranty terms |
| Deployment    | Rack, image, configure, baseline                    |
| Usage         | Patch, monitor, capacity-trend, change-manage       |
| End of life   | De-commission, secure wipe (Lab 25)                 |
| Disposal/recycle | Certified e-waste, audit trail, asset retirement |

Track which stage each asset is in inside the spreadsheet.

## Step 3 — Documentation set the exam expects

Build (or link to) at least these documents per server:

1. **Service manual** — vendor PDF link.
2. **Architecture diagram** — high-level component layout (use draw.io / Excalidraw).
3. **Infrastructure diagram** — network + storage paths.
4. **Workflow diagram** — request → approval → deploy.
5. **Recovery processes** — runbook for power, OS, app failure.
6. **Baselines** — file from Lab 12.
7. **Change management log** — ticket / RFC numbers.
8. **Server configuration** — the running config exported.

Capture the running config as a starting "as-built" doc:

```bash
mkdir -p /root/asbuilt && cd /root/asbuilt
cp -p /etc/fstab fstab.txt
ip -j addr show > ip-addr.json
systemctl list-units --type=service --state=running > services.txt
ufw status numbered > firewall.txt 2>/dev/null
crontab -l > crontab.txt 2>/dev/null
ls
```

## Step 4 — BIA-driven targets (MTBF, MTTR, RPO, RTO, SLA, uptime)

| Metric  | Means                                                   | Driven by                     |
|---------|----------------------------------------------------------|-------------------------------|
| **BIA** | Business Impact Analysis                                 | Stakeholders                  |
| **MTBF**| Mean Time Between Failures (reliability)                 | Hardware specs, history       |
| **MTTR**| Mean Time To Recover                                     | Runbook quality, spares       |
| **RPO** | Recovery **Point** Objective — max acceptable data loss  | Backup frequency (Lab 26)     |
| **RTO** | Recovery **Time** Objective — max acceptable downtime    | DR plan (Lab 27)              |
| **SLA** | Service Level Agreement — contractual uptime %           | Above + redundancy (Lab 14)   |
| **Uptime** | "Five nines" = 99.999 % ≈ 5.26 min downtime / year     | Cluster + DR + change mgmt    |

Uptime maths cheat:

| Uptime %  | Annual downtime |
|-----------|-----------------|
| 99 %      | 3 d 15 h 36 m   |
| 99.9 %    | 8 h 45 m 36 s   |
| 99.99 %   | 52 m 33 s       |
| 99.999 %  | 5 m 15 s        |

## Step 5 — Change management entry (template)

```text
RFC-2026-0521-001
System:         srv01 (asset tag SRV01-2024)
Change window:  2026-05-21 22:00–23:00 SGT
Risk:           Low
Implementer:    alice
Approver:       Bob (CAB)
Rollback:       LVM snapshot lv_root_pre_patch
Description:    Apply unattended-upgrades 22.04.x security set
```

Keep these alongside the asset record.

## Step 6 — Secure storage of sensitive documentation

Architecture diagrams and recovery runbooks should never be world-readable. Apply Server+ 2.7 guidance:

```bash
chmod 750 /root/asbuilt
chown root:root /root/asbuilt -R
```

Off-site copy belongs in an encrypted, access-controlled store (the same place as backup encryption keys — Lab 26/27).

## Step 7 — Licensing concepts cheat sheet

| Model               | When it's billed                       |
|---------------------|----------------------------------------|
| Per-instance        | Per running OS / app instance          |
| Per-concurrent user | Max simultaneous users                 |
| Per-server          | Per physical or virtual server         |
| Per-socket          | Per populated CPU socket               |
| Per-core            | Per physical core (Microsoft, Oracle)  |
| Site-based          | Whole site, unlimited within it        |
| Subscription        | Recurring fee, includes updates        |
| Volume licensing    | Discounted bulk pre-purchase           |
| Open source         | No licence cost; obligations per OSS licence (GPL, BSD, MIT…) |
| Node-locked         | Bound to a specific machine ID         |

Other terms to recognise:
- **Signatures** — cryptographic proof of a licence file's authenticity.
- **License vs. maintenance & support** — separate line items; you can keep using software whose support has lapsed but no patches.
- **True-up** — annual reconciliation of "what you actually used" vs. "what you bought".
- **Version compatibility** — backward compatible (new licence honours old version) vs. forward compatible (old licence honours new version).
- **Physical vs. virtual** — many enterprise licences require declaring whether a host is bare-metal or a VM (often costs differ).

## Step 8 — License count validation script

```bash
cat > /root/license-count.sh <<'EOF'
#!/usr/bin/env bash
echo "Physical sockets: $(lscpu | awk '/Socket\(s\)/{print $2}')"
echo "Physical cores:   $(lscpu | awk '/^Core\(s\) per socket/{c=$NF} /Socket\(s\)/{s=$NF} END{print c*s}')"
echo "vCPUs:            $(nproc)"
echo "OS instance count on this host: 1"
EOF
chmod +x /root/license-count.sh && /root/license-count.sh
```

Use the output during a **true-up** reconciliation.

---

## Free online tools
- **GLPI (open-source ITAM)** — https://glpi-project.org/
- **Snipe-IT (open-source asset tracking)** — https://snipeitapp.com/
- **draw.io / diagrams.net** — https://app.diagrams.net/
- **Excalidraw** — https://excalidraw.com/
- **CompTIA BCP/DR templates** — https://www.ready.gov/business/implementation

## What you learned
- A scripted asset-discovery pattern that fills the inventory spreadsheet.
- The eight documentation artefacts Server+ 2.7 requires.
- BIA-driven metrics: MTBF, MTTR, RPO, RTO, SLA, and the uptime maths.
- A change-management entry template you can carry to interview / exam questions.
- Why sensitive docs need restricted storage.
- The full Server+ 2.8 licensing model table and a quick true-up validation script.
