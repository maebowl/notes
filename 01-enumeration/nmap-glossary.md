# nmap — flag glossary

> every flag and option, explained. click a letter to jump around.

[a](#-a) · [b](#-b-ftp-relay-host) · [d](#-d-decoys) · [e](#-e-interface) · [f](#-f) · [i](#-il-file) · [n](#-n) · [o](#-o) · [p](#-p-ports) · [s](#-s-ip) · [t](#-t-0-5) · [v](#-v) · [-](#--disable-arp-ping)

---

## scan types

### -sS
**TCP SYN scan (half-open).** sends a SYN, waits for a response, but never completes the handshake. fast, stealthy, and the default when running as root. needs root privileges because it creates raw TCP packets.

### -sT
**TCP connect scan.** completes the full three-way handshake. more accurate but less stealthy — creates logs and is easily detected by IDS/IPS. the default when you're *not* root. sometimes called the "polite" scan because it behaves like a normal client connection.

### -sA
**TCP ACK scan.** sends packets with only the ACK flag. doesn't tell you if a port is open, but it *does* tell you if it's being filtered by a firewall. harder for firewalls to block because ACK packets look like they belong to an existing connection.

### -sU
**UDP scan.** scans UDP ports. much slower than TCP scans because UDP is stateless — no handshake, no acknowledgment, longer timeouts.

### -sW
**TCP window scan.** similar to ACK scan but examines the TCP window field of RST packets to determine if ports are open or closed.

### -sM
**TCP maimon scan.** sends FIN/ACK packets. some systems drop these for open ports, making it possible to distinguish open from closed.

### -sN
**TCP null scan.** sends a packet with no flags set. open ports typically don't respond; closed ports send RST.

### -sF
**TCP FIN scan.** sends a packet with only the FIN flag. similar logic to null scan.

### -sX
**TCP xmas scan.** sends a packet with FIN, PSH, and URG flags set (lit up like a christmas tree, hence the name). same logic as null and FIN scans.

### -sO
**IP protocol scan.** determines which IP protocols (TCP, UDP, ICMP, etc.) are supported by the target.

### -sI \<zombie host\>
**idle scan.** uses a "zombie" host to scan the target indirectly. extremely stealthy since the scan appears to come from the zombie, not you.

### -sY
**SCTP INIT scan.** like SYN scan but for SCTP protocol.

### -sZ
**SCTP COOKIE-ECHO scan.** more advanced SCTP scan.

### -sV
**service version detection.** probes open ports to determine what software and version is running. works by grabbing banners and using signature matching.

### -sC
**default script scan.** runs nmap's default set of NSE scripts against the target. equivalent to `--script default`.

---

## host discovery

### -sn
**skip port scanning.** only does host discovery — figures out which hosts are alive without scanning any ports. used to be called `-sP` (ping scan).

### -Pn
**skip host discovery.** treats all hosts as alive and goes straight to scanning. useful when you know the host is up but ICMP is blocked.

### -PE
**ICMP echo request.** sends a ping to check if the host is alive. the classic "are you there?" packet.

### -PU
**UDP ping.** sends a UDP packet to discover hosts.

### -PA
**TCP ACK ping.** sends a TCP ACK packet for host discovery.

### -PS
**TCP SYN ping.** sends a TCP SYN packet for host discovery.

---

## port specification

### -p \<ports\>
**specify ports to scan.** you can define them a bunch of ways:
- individual: `-p 22,80,443`
- range: `-p 22-445`
- all ports: `-p-`
- specific protocol: `-p U:53,T:80` (UDP 53, TCP 80)

### -F
**fast scan.** only scans the top 100 most common ports instead of the default 1000.

### --top-ports=\<n\>
**scan top N ports.** scans the N most commonly used ports based on nmap's frequency database.

---

## output

### -oN \<file\>
**normal output.** saves results in human-readable format (`.nmap`).

### -oG \<file\>
**grepable output.** saves results in a format that's easy to parse with grep, awk, and cut (`.gnmap`).

### -oX \<file\>
**XML output.** saves results as structured XML data (`.xml`). can be converted to HTML with `xsltproc`.

### -oA \<basename\>
**all formats.** saves results in normal, grepable, *and* XML at once. the move. just always use this.

---

## service and OS detection

### -A
**aggressive scan.** combines service detection (`-sV`), OS detection (`-O`), traceroute (`--traceroute`), and default scripts (`-sC`). one flag to rule them all.

### -O
**OS detection.** tries to determine the target's operating system by analyzing TCP/IP fingerprints.

---

## verbosity and debugging

### -v
**verbose.** shows more detailed output during the scan, including open ports as they're discovered in real time. use `-vv` for even more detail.

### --reason
**show reasoning.** tells you *why* nmap classified a port or host the way it did (e.g., `syn-ack`, `arp-response`, `no-response`).

### --packet-trace
**show all packets.** displays every packet sent and received during the scan. great for understanding exactly what nmap is doing under the hood.

### --stats-every=\<time\>
**periodic status updates.** shows scan progress at the specified interval (e.g., `--stats-every=5s` or `--stats-every=1m`).

---

## performance

### -T \<0-5\>
**timing template.** presets that control scan speed and aggressiveness:

| value | name | use case |
|---|---|---|
| `-T 0` | paranoid | IDS evasion, painfully slow |
| `-T 1` | sneaky | IDS evasion, still slow |
| `-T 2` | polite | low bandwidth, won't overwhelm target |
| `-T 3` | normal | the default |
| `-T 4` | aggressive | fast, reliable network assumed |
| `-T 5` | insane | maximum speed, may miss things |

### --min-rate \<number\>
**minimum packet rate.** tells nmap to send at least this many packets per second.

### --max-retries \<number\>
**max retry attempts.** how many times nmap will resend a probe to an unresponsive port (default: 10).

### --initial-rtt-timeout \<time\>
**initial round-trip timeout.** how long nmap waits for the first response from a port.

### --max-rtt-timeout \<time\>
**max round-trip timeout.** the absolute maximum time nmap will wait for a response.

---

## firewall/IDS evasion

### -D \<decoys\>
**decoy scan.** generates fake source IPs to hide your real one. use `RND:5` for 5 random decoys, or specify IPs manually. the decoy IPs should be alive to avoid triggering SYN-flood protections.

### -S \<IP\>
**spoof source IP.** makes the scan appear to come from a different IP address. useful for testing firewall rules from different subnets.

### -e \<interface\>
**specify network interface.** tells nmap which network interface to send packets through. required when using `-S` (source IP spoofing) so nmap knows *where* to send the packets from.

**how to find your interfaces:**

on linux:
```bash
ip a
```

on windows:
```cmd
ipconfig
```

common interface names:
- `eth0` — wired ethernet (your physical connection)
- `wlan0` — wireless
- `tun0` — VPN tunnel (this is the one you'll use on HTB/THM when connected to their VPN)
- `lo` — loopback (localhost, 127.0.0.1 — you probably don't want this one)

the trick is picking the right one. if you're scanning through a VPN, use `tun0`. if you're on a local network, use `eth0` or `wlan0`. if you pick the wrong interface, your packets go nowhere and nmap just sits there confused.

**quick way to figure it out:** look at which interface has an IP address in the same network as your target. if your target is `10.129.2.28` and `tun0` has `10.10.14.2`, that's your VPN — use `tun0`.

### --source-port \<port\>
**spoof source port.** makes the scan appear to come from a specific port. port 53 (DNS) is a classic choice because firewalls often trust DNS traffic.

### --disable-arp-ping
**disable ARP.** forces nmap to use ICMP instead of ARP for host discovery on local networks.

---

## scripting (NSE)

### --script \<scripts\>
**run specific scripts.** can be a script name, category, or comma-separated list:
- `--script banner` — a specific script
- `--script vuln` — a whole category
- `--script banner,smtp-commands` — multiple scripts

---

## DNS

### -n
**disable DNS resolution.** skips reverse DNS lookups. speeds things up and reduces noise.

### --dns-server \<ns\>
**custom DNS servers.** use specific DNS servers instead of the system defaults. useful in a DMZ where internal DNS servers are more trusted.

**how to find DNS servers to use:**

on linux:
```bash
# check your current DNS config
cat /etc/resolv.conf

# or if you're using systemd
resolvectl status
```

on windows:
```cmd
ipconfig /all
```
look for "DNS Servers" under your active network adapter.

**which DNS server should you actually use?**

- **target's own DNS server** — if you've discovered one during enumeration, this is gold. internal DNS servers know about internal hostnames that public DNS doesn't. if the target is running DNS on port 53, try using *that* as your DNS server.
- **the network's default DNS** — whatever's in `/etc/resolv.conf` or `ipconfig /all`. these are usually the ones the network trusts.
- **public DNS** (8.8.8.8, 1.1.1.1) — useful as a fallback, but won't resolve internal hostnames.

**why this matters:** in a pentest, the company's internal DNS servers often resolve hostnames that reveal internal infrastructure — subdomains, internal services, dev servers. public DNS won't show you any of that. using `--dns-server` with an internal DNS server can uncover hosts you'd otherwise never find.

---

## misc

### -b \<FTP relay host\>
**FTP bounce scan.** uses an FTP server to scan another host indirectly. old-school technique.

### --scanflags \<flags\>
**custom TCP flags.** lets you craft your own TCP flag combinations for custom scan types.

### -iL \<file\>
**input from list.** reads target hosts from a file instead of specifying them on the command line.
