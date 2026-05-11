# Lab 32 — Network Connectivity Troubleshooting

Maps to CompTIA Server+ objective **4.5 — Given a scenario, troubleshoot network connectivity issues** (lack of internet, resource unavailable, wrong DHCP info, host unreachable, unknown host, can't reach subnets, ISP failure, DNS/DHCP failure, misconfigured hosts file; causes — improper IP, IPv4 vs. IPv6, VLAN misconfig, port security, component failure, wrong OS routes, bad cables, firewall, misconfigured NIC; tools — check link lights, power, cables, commands: `ipconfig`, `ip addr`, `ping`, `tracert`, `traceroute`, `nslookup`, `netstat`, `dig`, `telnet`, `nc`, `nbtstat`, `route`).

Run all commands on https://killercoda.com/playgrounds/scenario/ubuntu

**Required software (free, apt):** `iproute2`, `iputils-ping`, `dnsutils`, `tcpdump`, `mtr-tiny`, `traceroute`, `nmap`, `netcat-openbsd`

```bash
apt update && apt install -y iproute2 iputils-ping dnsutils tcpdump mtr-tiny traceroute nmap netcat-openbsd
```

---

## Step 1 — Layer 1: link lights / cable / NIC up

```bash
ip -br link
ip -s link show eth0
ethtool eth0 2>/dev/null | head      # Speed, Duplex, Link detected
```

`Link detected: no` = cable / port / NIC fault. In the field: check the **link light** on the switch and on the NIC. If both dark → cable or port. Swap cable, then try a different switch port.

## Step 2 — Layer 2: NIC config and ARP

```bash
ip -br addr
ip neigh                          # ARP table — who answers on the LAN
```

If `ip neigh` shows `FAILED` for the gateway, the gateway is unreachable at layer 2 (cable / VLAN / port-security issue).

## Step 3 — Layer 3: addressing, gateway, route

```bash
ip route
ip route get 8.8.8.8
ip route get 10.99.99.50
```

Server+ 4.5 causes: **improper IP**, **wrong OS route tables**, **APIPA (169.254.x.x)** → DHCP failure (Lab 8).

Add / remove routes for testing:

```bash
ip route add 10.50.0.0/24 via 10.99.99.1 2>/dev/null
ip route del 10.50.0.0/24 2>/dev/null
```

Windows equivalents: `route print`, `route add`, `route delete`.

## Step 4 — Reachability: ping, traceroute, mtr

```bash
ping -c 3 127.0.0.1
ping -c 3 8.8.8.8 || echo "(some Killercoda playgrounds block ICMP outbound)"
traceroute -n 8.8.8.8 2>/dev/null | head
mtr -rwc 5 8.8.8.8 2>/dev/null | head
```

Decision tree:
- `127.0.0.1` fails → IP stack itself broken.
- Default gateway fails → layer 2 / VLAN / cable.
- Public IP fails but gateway OK → NAT / ISP / firewall.

## Step 5 — Name resolution: DNS chain

```bash
cat /etc/resolv.conf
dig +short example.com
dig example.com
nslookup example.com
host example.com
```

Symptom mapping:

| Symptom              | Likely cause                                |
|----------------------|----------------------------------------------|
| "Unknown host"       | DNS server unreachable or wrong             |
| Wrong IP returned    | Stale cache, hosts file override (Lab 8)    |
| `dig` works, app doesn't | App caches DNS or uses IPv6 first      |
| `nslookup` slow      | Reverse DNS timing out                      |

```bash
grep -E "^[^#].*(example|app|web)" /etc/hosts   # hosts file overrides
systemd-resolve --statistics 2>/dev/null | head
```

## Step 6 — Port reachability: telnet / nc / nmap

```bash
nc -vz 1.1.1.1 443 2>&1 | head
nc -vz example.com 80 2>&1 | head
nmap -p 22,80,443 -Pn 127.0.0.1 | head -20
```

Use `nc -vz` (or `telnet`) to test a single port. Use `nmap` for ranges. `telnet` is still on the exam.

## Step 7 — DHCP issues

```bash
journalctl -u systemd-networkd -n 30 --no-pager 2>/dev/null
journalctl -u NetworkManager  -n 30 --no-pager 2>/dev/null
ip -4 addr show eth0 | grep -q '169\.254' && echo "APIPA — DHCP failed"
```

Common DHCP failures from objective 4.5: server down, wrong scope, scope exhausted, **server misconfigured**, switch-port not on the right VLAN to reach the DHCP relay.

## Step 8 — Packet capture for the hard cases

```bash
tcpdump -ni eth0 -c 10 'icmp or port 53' 2>&1 | head
```

If you can't tell whether a request even leaves the host → tcpdump answers. If outbound is seen but inbound is silent → firewall / NAT / switch ACL is dropping the reply.

## Step 9 — Open / listening sockets (netstat / ss)

```bash
ss -tulnp
ss -tnp state established | head
netstat -tulnp 2>/dev/null | head
```

`nbtstat` (Windows) — the SMB / NetBIOS troubleshooting cousin. Linux equivalent is `nmblookup`.

## Step 10 — IPv4 vs. IPv6 surprises

```bash
ip -6 addr
getent ahosts example.com | head        # what the resolver returns
curl -s4 ifconfig.me; echo
curl -s6 ifconfig.me 2>/dev/null
```

If an app prefers IPv6 but the IPv6 path is broken, it appears as a **slow** or **partial** outage. Fix: either fix IPv6 routing or force the app to IPv4 (e.g. `curl -4`, `ssh -4`).

## Step 11 — VLAN, port security, and NIC config (causes)

| Cause                       | Indicator                                  |
|-----------------------------|---------------------------------------------|
| Misconfigured VLAN          | Host has IP but cannot reach gateway        |
| Switch port-security        | Link comes up then drops within seconds     |
| Misconfigured NIC (speed/duplex mismatch) | High error counters in `ip -s link` |
| Wrong subnet mask           | Local hosts reachable, distant hosts not    |

## Step 12 — Decision tree (the one you'll use on the exam)

```text
Symptom: user can't reach server X
  1. Check link lights on both ends                        (layer 1)
  2. ip addr / ipconfig — is there a valid IP?              (layer 3, DHCP)
  3. ping the default gateway                               (layer 2/3)
  4. ping a public IP (8.8.8.8)                             (NAT/ISP)
  5. dig / nslookup the target FQDN                          (DNS)
  6. ping the target IP                                      (routing/firewall)
  7. nc -vz target 443                                       (port/firewall)
  8. tcpdump for the silent direction                        (drop point)
```

Always work **bottom-up** through the OSI stack.

---

## Free online tools
- **subnet-calculator.com** — https://www.subnet-calculator.com/
- **iperf3 (bandwidth test)** — https://iperf.fr/
- **Wireshark (free packet analyser)** — https://www.wireshark.org/
- **dnsviz.net (DNS chain visualiser)** — https://dnsviz.net/

## What you learned
- A layer-by-layer triage of "I can't reach the server" — exactly the model the SK0-005 exam tests.
- Tool ↔ symptom mapping: `ip`, `ping`, `traceroute`, `mtr`, `dig`, `nslookup`, `nc`, `ss`, `tcpdump`.
- DHCP failures show up as APIPA; DNS failures show up as "unknown host".
- IPv4 vs. IPv6 dual-stack failure modes.
- VLAN, port-security, and NIC-config indicators.
- A repeatable bottom-up decision tree.
