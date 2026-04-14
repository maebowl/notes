# nmap — firewall and IDS/IPS evasion

> sometimes the target is protected. here's how to work around that.

---

## what are we dealing with?

### firewalls

software that monitors network traffic and decides what to allow or block based on rules. it checks individual packets and either **passes**, **ignores**, or **blocks** them.

### IDS (intrusion detection system)

*passively* monitors all traffic, looking for patterns that match known attacks. if it spots something, it alerts the admin. it doesn't block anything on its own.

### IPS (intrusion prevention system)

like IDS but with teeth — it can *automatically* take action to prevent detected attacks. IPS complements IDS. both use pattern matching and signatures to detect things like service detection scans.

---

## figuring out firewall rules

remember: when a port is filtered, it's either being **dropped** (no response) or **rejected** (explicit response with RST or ICMP error).

ICMP error types you might see:
- net unreachable
- net prohibited
- host unreachable
- host prohibited
- port unreachable
- proto unreachable

### the ACK scan trick (-sA)

the ACK scan is **much harder for firewalls to filter** than SYN or connect scans. why? because firewalls usually block incoming SYN packets (new connections from outside), but ACK packets look like they belong to an *existing* connection. the firewall can't easily tell if the connection started internally or externally.

#### SYN scan vs ACK scan — same ports, different results:

**SYN scan:**
```bash
sudo nmap 10.129.2.28 -p 21,22,25 -sS -Pn -n --disable-arp-ping --packet-trace
```
```
PORT   STATE    SERVICE
21/tcp filtered ftp
22/tcp open     ssh
25/tcp filtered smtp
```

**ACK scan:**
```bash
sudo nmap 10.129.2.28 -p 21,22,25 -sA -Pn -n --disable-arp-ping --packet-trace
```
```
PORT   STATE      SERVICE
21/tcp filtered   ftp
22/tcp unfiltered ssh
25/tcp filtered   smtp
```

notice: the SYN scan shows port 22 as **open** (got a SYN-ACK back), while the ACK scan shows it as **unfiltered** (got an RST back). for ports 21 and 25, both scans show filtered — those packets are getting dropped regardless.

the ACK scan doesn't tell you if a port is *open*, but it tells you if it's *being filtered*. super useful for mapping firewall rules.

---

## detecting IDS/IPS

this is harder because IDS/IPS are passive — they're just watching. here's the approach:

1. use multiple VPS servers with different IP addresses
2. if one gets blocked, you know there's active monitoring
3. switch to a different VPS and keep going
4. adjust your approach to be quieter

if you scan too aggressively from one IP and suddenly lose access, that's a pretty good sign an IPS kicked in.

---

## decoys (-D)

decoys make it look like the scan is coming from multiple IP addresses, hiding your real one in the crowd:

```bash
sudo nmap 10.129.2.28 -p 80 -sS -Pn -n --disable-arp-ping --packet-trace -D RND:5
```

```
SENT TCP 102.52.161.59:59289 > 10.129.2.28:80 S ...
SENT TCP 10.10.14.2:59289 > 10.129.2.28:80 S ...      <-- our real IP
SENT TCP 210.120.38.29:59289 > 10.129.2.28:80 S ...
SENT TCP 191.6.64.171:59289 > 10.129.2.28:80 S ...
SENT TCP 184.178.194.209:59289 > 10.129.2.28:80 S ...
SENT TCP 43.21.121.33:59289 > 10.129.2.28:80 S ...
```

`-D RND:5` generates 5 random IPs and mixes your real IP in randomly. the target sees 6 different "sources" and can't easily tell which one is actually you.

**important:** the decoy IPs need to be alive, or you risk triggering SYN-flood protections.

works with: SYN, ACK, ICMP scans, and OS detection.

---

## spoofing source IP (-S)

sometimes specific subnets are blocked. you can try scanning from a different source IP to test firewall rules:

```bash
# blocked — port shows filtered
sudo nmap 10.129.2.28 -n -Pn -p445 -O

PORT    STATE    SERVICE
445/tcp filtered microsoft-ds
```

```bash
# spoofed source IP — port is open!
sudo nmap 10.129.2.28 -n -Pn -p 445 -O -S 10.129.2.200 -e tun0

PORT    STATE SERVICE
445/tcp open  microsoft-ds
```

the firewall was blocking our subnet but allowing `10.129.2.200`. `-S` sets the source IP, `-e` specifies the network interface to use.

---

## DNS proxying

nmap does reverse DNS lookups by default — and most firewalls allow DNS traffic through because, well, the internet needs DNS to work. we can exploit this:

### using --source-port 53

if the firewall trusts traffic from DNS port 53, we can make our scans *look* like DNS traffic:

```bash
# normal scan — filtered
sudo nmap 10.129.2.28 -p50000 -sS -Pn -n --disable-arp-ping --packet-trace

PORT      STATE    SERVICE
50000/tcp filtered ibm-db2
```

```bash
# from source port 53 — open!
sudo nmap 10.129.2.28 -p50000 -sS -Pn -n --disable-arp-ping --packet-trace --source-port 53

PORT      STATE SERVICE
50000/tcp open  ibm-db2
```

the firewall is letting port 53 traffic through without proper filtering. once you find this, you can use it with netcat too:

```bash
ncat -nv --source-port 53 10.129.2.28 50000

Ncat: Connected to 10.129.2.28:50000.
220 ProFTPd
```

### using custom DNS servers

you can also specify which DNS servers to use with `--dns-server`. this is especially useful in a DMZ — the company's internal DNS servers are usually more trusted and might give you access to internal hosts.

---

## evasion options at a glance

| option | what it does |
|---|---|
| `-sA` | ACK scan — harder for firewalls to filter |
| `-D RND:5` | generate 5 random decoy IPs |
| `-S <IP>` | spoof source IP address |
| `-e <interface>` | specify network interface |
| `--source-port 53` | make traffic look like it's from DNS |
| `--dns-server <ns>` | use custom DNS servers |

---

## key takeaways

- ACK scans map firewall rules without telling you if ports are open
- decoys hide your real IP in a crowd of fake ones
- source port 53 is a classic firewall bypass — admins often forget to filter it properly
- always have backup VPS IPs ready in case IPS blocks you
- the goal is to be quiet and blend in with normal traffic
