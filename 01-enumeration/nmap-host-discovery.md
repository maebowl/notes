# nmap — host discovery

> before you scan ports, you gotta figure out who's even online.

---

## the idea

when you're doing an internal pentest on a company's network, step one is figuring out which systems are actually alive. nmap has a bunch of host discovery options for this, but the most effective method is **ICMP echo requests** (basically, pinging stuff).

**important:** always save every scan. you'll want them later for comparison, documentation, and reporting. different tools give different results, so having records matters.

---

## wait — how do i find my network range?

before you can scan a network, you need to know what network you're on. here's how:

### linux

```bash
ip a
```

look for your active interface (usually `eth0`, `wlan0`, or `tun0` for VPN). you'll see something like:

```
inet 10.10.14.2/24
```

that `/24` is your subnet — it means the network range is `10.10.14.0/24` (the first three octets stay the same, last one is 0-255, giving you 256 addresses).

### windows

```cmd
ipconfig
```

look for your IPv4 address and subnet mask. a subnet mask of `255.255.255.0` = `/24`.

### quick CIDR cheatsheet

| CIDR | subnet mask | hosts |
|---|---|---|
| `/24` | 255.255.255.0 | 256 |
| `/16` | 255.255.0.0 | 65,536 |
| `/8` | 255.0.0.0 | 16,777,216 |

so if your IP is `10.129.2.18/24`, your network range is `10.129.2.0/24`. that's what you feed to nmap.

---

## scanning a network range

```bash
sudo nmap 10.129.2.0/24 -sn -oA tnet | grep for | cut -d" " -f5
```

| option | what it does |
|---|---|
| `10.129.2.0/24` | target network range |
| `-sn` | disables port scanning (host discovery only) |
| `-oA tnet` | saves results in all formats as 'tnet' |

this only works if the hosts' firewalls allow ICMP. if they don't, you'll need other techniques (covered in firewall evasion).

## scanning from an IP list

sometimes during a pentest you'll get handed a list of IPs to test. nmap can work with that directly.

```bash
# the list
cat hosts.lst
10.129.2.4
10.129.2.10
10.129.2.11
10.129.2.18
10.129.2.19
10.129.2.20
10.129.2.28
```

```bash
sudo nmap -sn -oA tnet -iL hosts.lst | grep for | cut -d" " -f5
```

| option | what it does |
|---|---|
| `-iL hosts.lst` | reads targets from the list |

in this example, only 3 of 7 hosts came back as active. that doesn't necessarily mean the others are down — their firewalls might just be ignoring the default ICMP echo requests. nmap marks those as inactive since it got no response.

## scanning multiple IPs

if you only need a few specific IPs, just list them:

```bash
sudo nmap -sn -oA tnet 10.129.2.18 10.129.2.19 10.129.2.20
```

or if they're sequential, use a range:

```bash
sudo nmap -sn -oA tnet 10.129.2.18-20
```

---

## scanning a single host

```bash
sudo nmap 10.129.2.18 -sn -oA host
```

with `-sn`, nmap automatically does a ping scan with ICMP echo requests (`-PE`). but here's the interesting part — if you're on the same network, nmap actually sends an **ARP ping first** before the ICMP request, and uses the ARP reply to determine if the host is up.

### seeing what's actually happening

use `--packet-trace` to see the packets being sent:

```bash
sudo nmap 10.129.2.18 -sn -oA host -PE --packet-trace
```

```
SENT (0.0074s) ARP who-has 10.129.2.18 tell 10.10.14.2
RCVD (0.0309s) ARP reply 10.129.2.18 is-at DE:AD:00:00:BE:EF
```

see? it used ARP, not ICMP. nmap took a shortcut.

### finding out *why* nmap thinks a host is alive

```bash
sudo nmap 10.129.2.18 -sn -oA host -PE --reason
```

the `--reason` flag tells you exactly why nmap marked a host as up (in this case: `arp-response`).

### forcing ICMP (disabling ARP)

if you actually want ICMP echo requests, disable ARP pings:

```bash
sudo nmap 10.129.2.18 -sn -oA host -PE --packet-trace --disable-arp-ping
```

```
SENT (0.0107s) ICMP [10.10.14.2 > 10.129.2.18 Echo request (type=8/code=0) id=13607 seq=0]
RCVD (0.0152s) ICMP [10.129.2.18 > 10.10.14.2 Echo reply (type=0/code=0) id=13607 seq=0]
```

*now* it's using ICMP. pay attention to these details — an ICMP echo request can help you not just confirm the host is alive, but also identify what system it's running.

---

## useful options at a glance

> click any flag to see its full explanation in the [glossary](nmap-glossary.md).

| option | what it does |
|---|---|
| [`-sn`](nmap-glossary.md#-sn) | host discovery only, no port scan |
| [`-PE`](nmap-glossary.md#-pe) | ICMP echo request |
| [`-oA <name>`](nmap-glossary.md#-oa-basename) | save results in all formats |
| [`-iL <file>`](nmap-glossary.md#-il-file) | read targets from a file |
| [`--packet-trace`](nmap-glossary.md#--packet-trace) | show all packets sent and received |
| [`--reason`](nmap-glossary.md#--reason) | show why a port/host is in a particular state |
| [`--disable-arp-ping`](nmap-glossary.md#--disable-arp-ping) | force ICMP instead of ARP on local networks |
