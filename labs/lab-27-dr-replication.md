# Lab 27 — Disaster Recovery: Replication and Site Failover

Maps to CompTIA Server+ objective **3.8 — Explain the importance of disaster recovery** (site types: hot, cold, warm, cloud, separate geographic locations; replication: constant, background, synchronous vs. asynchronous, application-consistent, file-locking, mirroring, bidirectional; testing: tabletops, live failover, simulated failover, production vs. non-production).

Run all commands on https://killercoda.com/playgrounds/scenario/ubuntu

**Required software (free, apt):** `rsync`, `lsyncd`, `keepalived`, `inotify-tools`

```bash
apt update && apt install -y rsync lsyncd keepalived inotify-tools
```

---

## Step 1 — Site types comparison

| Site type | Hardware at site | Data at site            | Time to bring up | Cost   |
|-----------|------------------|--------------------------|------------------|--------|
| **Hot**   | Running 24/7     | Replicated near-real-time | Minutes          | High   |
| **Warm**  | Provisioned, off | Restored from recent backup | Hours          | Medium |
| **Cold**  | Space + power only | Restored from off-site tape | Days           | Low    |
| **Cloud** | Provisioned on-demand | Replicated to cloud   | Minutes to hours | Variable |

DR sites must be in **separate geographic locations** — far enough that one disaster (flood, fire, regional outage) does not affect both.

## Step 2 — Asynchronous replication with rsync (background)

Simulate two sites with two directories:

```bash
mkdir -p /srv/primary /srv/dr
echo "v1" > /srv/primary/app.txt

# Background async replication every 60 s via cron / lsyncd
rsync -avh --delete /srv/primary/ /srv/dr/
ls /srv/dr/
```

**Asynchronous** = primary acknowledges the write before the replica has it. Cheaper, but the **RPO** = the replication lag.

## Step 3 — Near-synchronous replication with `lsyncd`

`lsyncd` watches the source with inotify and pushes deltas to the target within seconds:

```bash
cat > /etc/lsyncd/lsyncd.conf.lua <<'EOF'
settings { logfile = "/var/log/lsyncd.log", statusFile = "/var/run/lsyncd.status" }
sync {
    default.rsync,
    source = "/srv/primary",
    target = "/srv/dr",
    delay  = 2
}
EOF
systemctl restart lsyncd
echo "v2" > /srv/primary/app.txt
sleep 4
cat /srv/dr/app.txt        # should print v2
```

## Step 4 — Synchronous replication concept

**Synchronous** replication blocks the write until **both** sites confirm — RPO = 0. Used for:

- **DRBD** (block-level replicated devices — Linux)
- Storage-array sync replication (NetApp SnapMirror sync, Dell SRDF)
- **MySQL semi-sync**, **PostgreSQL synchronous_commit = remote_apply**

Synchronous is **distance-bounded**: latency adds to every write. Practical limit is ~ 50–100 km.

## Step 5 — Constant vs. background

| Mode               | What it means                                            |
|--------------------|-----------------------------------------------------------|
| **Constant**       | Replication is always running — every change ships ASAP   |
| **Background**     | Periodic batches (e.g. 5-min rsync) — lower load, higher RPO |

Constant ≈ what `lsyncd` does. Background ≈ cron-driven rsync.

## Step 6 — Application-consistent replication and file locking

Server+ 3.8 lists **application consistent**, **file locking**, **mirroring**, **bidirectional**.

- **Application-consistent** = replica is the database in a state the DB will recover cleanly (flushed logs, no in-flight transactions). Done by quiescing the app, snapshotting, then shipping the snapshot.
- **File locking** = replication tool must respect locks so it does not capture half-written files.
- **Mirroring** = one-way primary → replica.
- **Bidirectional** = both sites can accept writes. Powerful but introduces **conflict resolution** that the storage / app must handle.

## Step 7 — VIP failover with keepalived (DR cut-over plumbing)

On a real DR pair, the application VIP floats. With `keepalived` (see Lab 14):

```text
PRIMARY  state MASTER  priority 150  → owns VIP 10.99.99.99
DR       state BACKUP  priority 100  → takes VIP if heartbeat fails
```

A failure of the master triggers BACKUP to **gratuitous-ARP** the VIP and take over within seconds. The application reconnects to the same VIP, now at the DR site.

## Step 8 — DR testing — Server+ lists four types

| Test                  | What happens                                         | Risk     |
|-----------------------|-------------------------------------------------------|----------|
| **Tabletop**          | Team walks through the runbook in a meeting room      | None     |
| **Simulated failover**| Failover into an isolated test environment            | Low      |
| **Live failover**     | Production traffic is moved to the DR site            | High     |
| **Production vs. non-production** | Decide which environment you'll test in — never your first live failover during a real outage | — |

Schedule: tabletop quarterly, simulated yearly, live every 1–2 years (or as audit demands).

## Step 9 — Tabletop DR exercise script (run this with the team)

```text
SCENARIO: Primary data centre has a building-wide power outage estimated 12 hours.
INJECTS:
  T+00m  Power lost. Monitoring alerts fire.
  T+05m  Confirm with facilities — outage real, ETA 12h.
  T+15m  Declare disaster. Open bridge call.
  T+30m  Begin DR runbook step 1: redirect DNS to DR site.
  T+45m  Promote DR DB replica to primary (app-consistent? yes/no — discuss).
  T+60m  Service back up at DR. Confirm RTO target.
  T+12h  Power restored. Begin fail-back: replicate writes back to primary, then cut over.
DECISIONS TO MAKE:
  - Who declares the disaster?
  - When do we communicate to customers?
  - When is the SLA breached?
```

Document every gap revealed in the exercise — those are the **action items**.

## Step 10 — RPO / RTO vs. replication choice

| Replication strategy        | Achievable RPO   | Cost   |
|-----------------------------|-------------------|--------|
| Tape off-site weekly        | 1 week           | Low    |
| Nightly rsync to cloud      | 24 hours         | Low    |
| `lsyncd` near-real-time     | ~10 seconds      | Medium |
| Sync block-level (DRBD/SRDF)| 0                | High   |

Pick the cheapest one that meets the **RPO from the BIA** (Lab 17). Buying more than required is wasted spend.

---

## Free online tools
- **NIST SP 800-34 (Contingency Planning)** — https://csrc.nist.gov/publications/detail/sp/800-34/rev-1/final
- **DRII Professional Practices** — https://drii.org/
- **lsyncd manual** — https://axkibe.github.io/lsyncd/
- **DRBD docs** — https://docs.linbit.com/

## What you learned
- Hot / warm / cold / cloud site comparison with cost and time-to-recover.
- Async (rsync, lsyncd) vs. sync (DRBD, SRDF, semi-sync DB) replication trade-offs.
- Application-consistent, file-locking, mirroring, bidirectional replication semantics.
- VIP failover plumbing with keepalived (cut-over mechanics).
- Four DR test types and a ready-to-run tabletop script.
- Matching the replication strategy to the BIA-driven RPO/RTO numbers.
