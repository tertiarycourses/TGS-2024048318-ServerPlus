# Lab 31 — OS and Software Troubleshooting

Maps to CompTIA Server+ objective **4.4 — Given a scenario, troubleshoot common OS and software problems** (unable to log on, unable to access resources/files, system file corruption, EoL/EoS, slow performance, service failures, hanging, freezing, patch update failure; causes — incompatible drivers, improperly applied patches, unstable drivers, server not joined to domain, clock skew, memory leaks, buffer overrun, incompatibility, insecure dependencies, version management, architecture, update failures, missing updates/dependencies, downstream failures, inappropriate app-level permissions, improper CPU affinity/priority; tools — patching upgrades/downgrades, package management, recovery boot modes — safe mode, single user mode, reload OS, snapshots, privilege escalation — runas, sudo, su, scheduled reboots, software firewalls, NTP/system time, services and processes — start/stop/identify/dependencies, configuration management — SCCM, Puppet/Chef/Ansible, GPO, HCL).

Run all commands on https://killercoda.com/playgrounds/scenario/ubuntu

**Required software (free, apt):** `systemd`, `strace`, `dpkg`, `chrony`, `ansible`

```bash
apt update && apt install -y strace chrony ansible
```

---

## Step 1 — systemd services: start, stop, status, dependencies

```bash
systemctl status sshd
systemctl list-dependencies sshd | head -20
systemctl list-units --type=service --state=failed
journalctl -u sshd -n 30 --no-pager
```

Service-failure pattern: status → recent logs → dependency check.

## Step 2 — Hanging / freezing service investigation

```bash
apt install -y nginx
systemctl start nginx
PID=$(systemctl show nginx -p MainPID --value)
ps -o pid,stat,cmd -p $PID
strace -p $PID -e trace=accept4,read,write -c -f -t -o /tmp/strace.log &
SP=$!; sleep 2; kill $SP 2>/dev/null
head /tmp/strace.log
```

Use `strace` only to **diagnose** — never leave it attached to a production process under load.

## Step 3 — Clock skew breaks authentication

Server+ 4.4 explicitly lists **clock skew** as a cause. Kerberos rejects auth if drift > 5 min.

```bash
chronyc tracking 2>/dev/null | head
timedatectl
date -u
```

Force a sync:

```bash
systemctl restart chrony
chronyc -a 'burst 4/4'
sleep 5
chronyc tracking | head
```

Tie-in: a CMOS battery failure (Lab 29) causes clock to reset on every boot → constant skew → users "randomly can't log in".

## Step 4 — Patch failure / improperly applied patch

Manufacture a stuck apt state:

```bash
apt install -y figlet
dpkg --status figlet | head -5
# Simulate breakage by removing a postinst-required file:
echo "broken" > /tmp/fake.deb
dpkg -i /tmp/fake.deb 2>&1 | head
dpkg --configure -a 2>&1 | head
apt -f install 2>&1 | head
```

Recovery flow: `dpkg --configure -a` → `apt -f install` → re-run failed unit. Server+ lists **patch update failure** as a symptom; the cause is usually a missed dependency or a held package.

## Step 5 — Missing dependencies / downstream failures

```bash
apt-cache rdepends openssl | head     # who depends on openssl
ldd /usr/sbin/sshd                     # what libs sshd needs
```

A patch that pulls a new libssl can cascade — `needrestart` (Lab 24) flags downstream services to restart.

## Step 6 — Recovery boot modes

| Mode                | Linux equivalent                              | When                                |
|---------------------|------------------------------------------------|--------------------------------------|
| **Safe mode**       | `systemd.unit=rescue.target`                  | Service won't let you boot           |
| **Single-user mode**| `systemd.unit=emergency.target` or `init=/bin/bash` | Total OS recovery — root password reset |
| **Reload OS**       | Reinstall from media                          | Corrupted boot beyond repair         |
| **Snapshots**       | LVM / VM snapshot (Lab 3 / 15) → revert       | Failed patch                         |

Show the GRUB edit hint to a student (won't actually reboot Killercoda):

```text
At the GRUB menu, press 'e' on the kernel line, append:
    systemd.unit=emergency.target
Press Ctrl-X to boot.
```

## Step 7 — Software firewall: adding/removing ports, zones

```bash
ufw status numbered
ufw allow 8443/tcp comment "app tier"
ufw delete allow 8443/tcp
```

"App can't reach DB after patch" is often "firewall service was reset to default by the upgrade". Always recheck rules post-patch.

## Step 8 — Privilege escalation troubleshooting

| Command       | Notes                                                 |
|---------------|--------------------------------------------------------|
| `sudo cmd`    | Run as root (or specified user) with policy from Lab 20 |
| `su -`        | Switch shell to root — requires root password         |
| `runas` (Win) | Windows equivalent of sudo                            |

```bash
sudo -l       # show what current user is allowed
sudo -k       # invalidate cached creds for safe test
```

## Step 9 — Memory leak / improper CPU affinity & priority

```bash
top -bn1 -o %MEM | head -15
ps -eo pid,comm,rss,nice,psr --sort=-rss | head -10
```

Symptoms: RSS grows unbounded over hours → memory leak → restart service, file bug, monitor.

Set CPU affinity (pin) and nice priority to isolate workloads:

```bash
PID=$(pgrep -n nginx) ; [ -n "$PID" ] && taskset -cp 0 $PID 2>/dev/null
renice -n 10 -p ${PID:-1} 2>/dev/null
```

## Step 10 — Package & version management

```bash
apt-cache policy nginx
apt-mark hold nginx          # pin to current version
apt-mark showhold
# Downgrade:
# apt install nginx=1.18.0-6ubuntu14.4 --allow-downgrades
```

**Version management** + **architecture** mismatches (e.g. installing `amd64` package on `arm64` host) are explicit causes in objective 4.4.

## Step 11 — Configuration management (the *prevention* tools)

Server+ 4.4 lists **SCCM, Puppet, Chef, Ansible, GPO** under tools. Demonstrate Ansible idempotency:

```bash
mkdir -p /srv/ansible && cd /srv/ansible
cat > playbook.yml <<'EOF'
- hosts: localhost
  connection: local
  become: false
  tasks:
    - name: Ensure /tmp/marker exists
      file: path=/tmp/marker state=touch mode=0644
    - name: Ensure motd content
      copy:
        dest: /tmp/motd-lab
        content: "managed by ansible at {{ ansible_date_time.iso8601 }}\n"
EOF
ansible-playbook playbook.yml 2>&1 | tail -10
ansible-playbook playbook.yml 2>&1 | tail -10   # second run = no changes (idempotent)
```

The take-away: drifted config is the #1 cause of "it worked yesterday". A configuration-management tool prevents drift.

## Step 12 — Scheduled reboots

```bash
# Linux:
# echo "0 3 * * 0 root /sbin/shutdown -r now" > /etc/cron.d/weekly-reboot
echo "Scheduled reboots are valid for systems with memory leaks or kernel-update accumulation"
```

Use **cluster-aware patching** (Lab 14) — never reboot all nodes simultaneously.

---

## Free online tools
- **systemd documentation** — https://systemd.io/
- **Ansible docs** — https://docs.ansible.com/
- **GNU strace docs** — https://strace.io/
- **Microsoft Group Policy Reference** — https://learn.microsoft.com/en-us/windows/win32/policy/

## What you learned
- systemd service triage workflow (`status` → `journalctl` → dependencies).
- Diagnosing hangs with `strace`.
- Clock skew → auth failures, and how `chrony` / NTP fixes it.
- Recovery from broken `dpkg` state and held / pinned packages.
- Boot recovery modes (rescue, emergency, reload OS, snapshot revert).
- Privilege-escalation paths and how to verify them (`sudo -l`).
- Memory leak symptoms; CPU affinity + nice priority controls.
- Configuration management with Ansible to prevent drift.
