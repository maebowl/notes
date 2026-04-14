# nmap — port scanning

> ok, so the host is alive. now what's running on it?

---

## port states

there are **6 possible states** for a scanned port:

| state | meaning |
|---|---|
| **open** | connection established — could be TCP, UDP, or SCTP |
| **closed** | port responded with RST flag. still useful — confirms the host is alive |
| **filtered** | nmap can't tell if it's open or closed. no response, or got an error code. probably a firewall |
| **unfiltered** | only shows up in TCP-ACK scans. port is accessible but can't determine open/closed |
| **open\|filtered** | no response — firewall or packet filter might be protecting it |
| **closed\|filtered** | only in IP ID idle scans. can't tell if closed or firewalled |

---

## TCP scans

### defining which ports to scan

| option | what it does |
|---|---|
| `-p 22,25,80,139,445` | specific ports |
| `-p 22-445` | port range |
| `--top-ports=10` | top N most common ports |
| `-p-` | *all* 65,535 ports |
| `-F` | fast scan — top 100 ports |

### SYN scan (-sS) — the default

this is the go-to. sends a SYN, never completes the handshake, so no full TCP connection is made. fast as hell — thousands of ports per second. needs root privileges.

if you're not root, nmap falls back to the connect scan (`-sT`) instead.

### connect scan (-sT) — the polite one

completes the full three-way handshake. more accurate, but way less stealthy — it creates logs on most systems and IDS/IPS will spot it easily. that said, it plays nice with services (behaves like a normal client connection), so it's less likely to crash anything.

```bash
sudo nmap 10.129.2.28 -p 443 --packet-trace --disable-arp-ping -Pn -n --reason -sT
```

```
CONN (0.0385s) TCP localhost > 10.129.2.28:443 => Operation now in progress
CONN (0.0396s) TCP localhost > 10.129.2.28:443 => Connected
PORT    STATE SERVICE REASON
443/tcp open  https   syn-ack
```

### tracing packets to understand what's happening

strip away all the extra noise with these flags to see *just* the scan:

```bash
sudo nmap 10.129.2.28 -p 21 --packet-trace -Pn -n --disable-arp-ping
```

| flag | purpose |
|---|---|
| `-Pn` | skip ICMP echo requests |
| `-n` | skip DNS resolution |
| `--disable-arp-ping` | skip ARP |

in the output, you'll see lines like:

```
SENT (0.0429s) TCP 10.10.14.2:63090 > 10.129.2.28:21 S ttl=56 id=57322 ...
RCVD (0.0573s) TCP 10.129.2.28:21 > 10.10.14.2:63090 RA ttl=64 id=0 ...
```

- **S** = SYN flag (we're initiating)
- **SA** = SYN-ACK (port is open)
- **RA** = RST-ACK (port is closed — target acknowledged and killed the connection)

---

## filtered ports — what's going on?

when a port shows as filtered, it's usually a firewall doing one of two things:

### 1. dropping packets (silent)

the firewall just swallows your packet. nmap gets nothing back, retries (default 10 times via `--max-retries`), and eventually gives up. this is *slow* — you'll notice the scan takes way longer.

```bash
sudo nmap 10.129.2.28 -p 139 --packet-trace -n --disable-arp-ping -Pn
```

```
SENT (0.0381s) TCP 10.10.14.2:60277 > 10.129.2.28:139 S ...
SENT (1.0411s) TCP 10.10.14.2:60278 > 10.129.2.28:139 S ...
PORT    STATE    SERVICE
139/tcp filtered netbios-ssn
```

two attempts, no response, 2+ seconds. that's a drop.

### 2. rejecting packets (loud)

the firewall sends back an ICMP "port unreachable" message. faster to detect, but same result — filtered.

```bash
sudo nmap 10.129.2.28 -p 445 --packet-trace -n --disable-arp-ping -Pn
```

```
SENT (0.0388s) TCP ... > 10.129.2.28:445 S ...
RCVD (0.0487s) ICMP [10.129.2.28 > ... Port 445 unreachable (type=3/code=3)]
PORT    STATE    SERVICE
445/tcp filtered microsoft-ds
```

if you *know* the host is alive, a reject strongly suggests the firewall is actively blocking this port. worth revisiting later.

---

## UDP scanning (-sU)

admins sometimes forget to filter UDP ports. oops.

UDP is stateless — no three-way handshake, no acknowledgment. so scans are **much slower** since timeouts are longer.

```bash
sudo nmap 10.129.2.28 -F -sU
```

nmap sends empty datagrams to UDP ports. here's how to read the responses:

| response | meaning |
|---|---|
| UDP response back | port is **open** (and the app is configured to respond) |
| ICMP type 3, code 3 | port is **closed** (unreachable) |
| no response | **open\|filtered** — could be either, can't tell |

```bash
# confirming an open UDP port
sudo nmap 10.129.2.28 -sU -Pn -n --disable-arp-ping --packet-trace -p 137 --reason
```

```
PORT    STATE SERVICE    REASON
137/udp open  netbios-ns udp-response ttl 64
```

```bash
# confirming a closed UDP port
sudo nmap 10.129.2.28 -sU -Pn -n --disable-arp-ping --packet-trace -p 100 --reason
```

```
PORT    STATE  SERVICE REASON
100/udp closed unknown port-unreach ttl 64
```

---

## useful options at a glance

> click any flag to see its full explanation in the [glossary](nmap-glossary.md).

| option | what it does |
|---|---|
| [`-sS`](nmap-glossary.md#-ss) | SYN scan (default with root) |
| [`-sT`](nmap-glossary.md#-st) | connect scan (full handshake) |
| [`-sU`](nmap-glossary.md#-su) | UDP scan |
| [`-p <ports>`](nmap-glossary.md#-p-ports) | specify ports |
| [`-p-`](nmap-glossary.md#-p-ports) | all ports |
| [`-F`](nmap-glossary.md#-f) | fast — top 100 |
| [`--top-ports=N`](nmap-glossary.md#--top-portsn) | top N most common |
| [`--packet-trace`](nmap-glossary.md#--packet-trace) | see all packets |
| [`-Pn`](nmap-glossary.md#-pn) | skip host discovery |
| [`-n`](nmap-glossary.md#-n) | skip DNS resolution |
| [`--reason`](nmap-glossary.md#--reason) | explain why a port is in its state |
| [`--disable-arp-ping`](nmap-glossary.md#--disable-arp-ping) | no ARP, use ICMP |
