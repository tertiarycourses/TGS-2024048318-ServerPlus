# Lab 8 — IP, VLAN, DNS, DHCP, FQDN, and Hosts File

Maps to CompTIA Server+ objective **2.2 — Given a scenario, configure servers to use network infrastructure services** (IP configuration, VLAN, default gateways, DNS, FQDN, hosts file, IPv4 RFC 1918, IPv6, firewall ports, DHCP, APIPA, MAC addresses).

Run all commands on https://killercoda.com/playgrounds/scenario/ubuntu

**Required software (free, apt):** `iproute2`, `vlan`, `bind9`, `dnsutils`, `isc-dhcp-server`

```bash
apt update && apt install -y iproute2 vlan bind9 bind9utils dnsutils isc-dhcp-server
```

---

## Step 1 — Inspect IP configuration (static vs. dynamic)

```bash
ip -c addr
ip -c route
ip link show | head
cat /etc/resolv.conf
```

Note the **MAC address** on each `link/ether` line and the **default gateway** (`default via …`).

## Step 2 — Configure a static IP and default gateway

```bash
ip addr add 10.99.99.10/24 dev eth0
ip route add default via 10.99.99.1 dev eth0 || true
ip -c addr show eth0
```

## Step 3 — VLAN tagging on a single NIC

```bash
modprobe 8021q
ip link add link eth0 name eth0.20 type vlan id 20
ip addr add 192.168.20.10/24 dev eth0.20
ip link set eth0.20 up
ip -d link show eth0.20
```

`eth0.20` carries VLAN 20 tags. On a real network the switch port must be **trunked** with VLAN 20 allowed.

## Step 4 — RFC 1918 private address spaces (memorise for the exam)

| Prefix          | Range                            | Default mask |
|-----------------|----------------------------------|--------------|
| 10.0.0.0/8      | 10.0.0.0 – 10.255.255.255        | /8           |
| 172.16.0.0/12   | 172.16.0.0 – 172.31.255.255      | /12          |
| 192.168.0.0/16  | 192.168.0.0 – 192.168.255.255    | /16          |

Plus: **169.254.0.0/16 = APIPA** — what a client self-assigns if DHCP fails.

## Step 5 — IPv6 quick check

```bash
ip -6 addr
sysctl net.ipv6.conf.all.disable_ipv6
ping6 -c2 ::1
```

Every interface gets at least a **link-local fe80::/10** address derived from the MAC.

## Step 6 — DNS server (BIND9) with a tiny zone

```bash
cat > /etc/bind/named.conf.local <<'EOF'
zone "lab.local" {
    type master;
    file "/etc/bind/db.lab.local";
};
EOF

cat > /etc/bind/db.lab.local <<'EOF'
$TTL 604800
@   IN  SOA ns1.lab.local. admin.lab.local. ( 2 604800 86400 2419200 604800 )
@   IN  NS  ns1.lab.local.
ns1 IN  A   127.0.0.1
web IN  A   10.99.99.20
db  IN  A   10.99.99.30
EOF

systemctl restart bind9
dig @127.0.0.1 web.lab.local +short          # → 10.99.99.20
dig @127.0.0.1 web.lab.local                  # full record
```

`web.lab.local` is the **FQDN** = hostname + domain + trailing dot.

## Step 7 — Hosts file (the local override that beats DNS)

```bash
echo "10.99.99.50  app.lab.local app" >> /etc/hosts
getent hosts app.lab.local
```

`/etc/hosts` (or `C:\Windows\System32\drivers\etc\hosts`) is consulted **before** DNS — the source of many "DNS issues" that aren't actually DNS.

## Step 8 — DHCP server on a private subnet

```bash
cat > /etc/dhcp/dhcpd.conf <<'EOF'
default-lease-time 600;
max-lease-time 7200;
authoritative;
subnet 10.99.99.0 netmask 255.255.255.0 {
  range 10.99.99.100 10.99.99.200;
  option routers 10.99.99.1;
  option domain-name-servers 127.0.0.1;
  option domain-name "lab.local";
}
EOF
# Don't actually start the daemon on Killercoda's shared net — just validate:
dhcpd -t -cf /etc/dhcp/dhcpd.conf && echo "Config OK"
```

The pool excludes `.1` (gateway) and `.2 – .99` (static reservations).

## Step 9 — APIPA simulation

```bash
ip addr add 169.254.42.42/16 dev eth0
ip -c addr show eth0
```

If a Windows client cannot reach a DHCP server it self-assigns from **169.254.0.0/16**. Seeing a 169.254.x.x address is a tell-tale that DHCP is failing.

---

## Free online tools
- **IP Calculator** — https://alfredang.github.io/ipcalculator/
- **IANA IPv4 special-use registry** — https://www.iana.org/assignments/iana-ipv4-special-registry/
- **BIND9 ARM** — https://bind9.readthedocs.io/

## What you learned
- Static IP, default gateway, MAC address, and the difference between DHCP and APIPA (169.254.0.0/16).
- VLAN tagging on a single NIC with 802.1Q.
- RFC 1918 private ranges you must memorise for SK0-005.
- IPv4 vs. IPv6 quick checks.
- Building a BIND9 zone with A records and an FQDN.
- How `/etc/hosts` short-circuits DNS — and why that matters when troubleshooting.
- A minimal ISC DHCP server scope.
