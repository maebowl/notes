# nmap — flag glossary

> every flag and option, explained. click a letter to jump around.

[a](#-a) · [b](#-b-ftp-relay-host) · [d](#-d-decoys) · [e](#-e-interface) · [f](#-f) · [i](#-il-file) · [n](#-n) · [o](#-o) · [p](#-p-ports) · [s](#-s-ip) · [t](#-t-0-5) · [v](#-v) · [-](#--disable-arp-ping)

---

## scan types

### -sS
**TCP SYN scan (half-open).** sends a SYN, waits for a response, but never completes the handshake. fast, stealthy, and the default when running as root. needs root privileges because it creates raw TCP packets.

```bash
sudo nmap -sS 10.129.2.28
sudo nmap -sS -p 22,80,443 10.129.2.28
```

### -sT
**TCP connect scan.** completes the full three-way handshake. more accurate but less stealthy — creates logs and is easily detected by IDS/IPS. the default when you're *not* root. sometimes called the "polite" scan because it behaves like a normal client connection.

```bash
nmap -sT 10.129.2.28
nmap -sT -p 443 10.129.2.28
```

### -sA
**TCP ACK scan.** sends packets with only the ACK flag. doesn't tell you if a port is open, but it *does* tell you if it's being filtered by a firewall. harder for firewalls to block because ACK packets look like they belong to an existing connection.

```bash
sudo nmap -sA -p 21,22,25 10.129.2.28
```

### -sU
**UDP scan.** scans UDP ports. much slower than TCP scans because UDP is stateless — no handshake, no acknowledgment, longer timeouts.

```bash
sudo nmap -sU 10.129.2.28
sudo nmap -sU -p 53,137,138 10.129.2.28
```

### -sW
**TCP window scan.** similar to ACK scan but examines the TCP window field of RST packets to determine if ports are open or closed.

```bash
sudo nmap -sW -p 22,80 10.129.2.28
```

### -sM
**TCP maimon scan.** sends FIN/ACK packets. some systems drop these for open ports, making it possible to distinguish open from closed.

```bash
sudo nmap -sM 10.129.2.28
```

### -sN
**TCP null scan.** sends a packet with no flags set. open ports typically don't respond; closed ports send RST.

```bash
sudo nmap -sN -p 80 10.129.2.28
```

### -sF
**TCP FIN scan.** sends a packet with only the FIN flag. similar logic to null scan.

```bash
sudo nmap -sF -p 80 10.129.2.28
```

### -sX
**TCP xmas scan.** sends a packet with FIN, PSH, and URG flags set (lit up like a christmas tree, hence the name). same logic as null and FIN scans.

```bash
sudo nmap -sX -p 80 10.129.2.28
```

### -sO
**IP protocol scan.** determines which IP protocols (TCP, UDP, ICMP, etc.) are supported by the target.

```bash
sudo nmap -sO 10.129.2.28
```

### -sI \<zombie host\>
**idle scan.** uses a "zombie" host to scan the target indirectly. extremely stealthy since the scan appears to come from the zombie, not you.

```bash
sudo nmap -sI zombie.example.com 10.129.2.28
sudo nmap -sI 10.129.2.100:80 10.129.2.28
```

### -sY
**SCTP INIT scan.** like SYN scan but for SCTP protocol.

```bash
sudo nmap -sY 10.129.2.28
```

### -sZ
**SCTP COOKIE-ECHO scan.** more advanced SCTP scan.

```bash
sudo nmap -sZ 10.129.2.28
```

### -sV
**service version detection.** probes open ports to determine what software and version is running. works by grabbing banners and using signature matching.

```bash
sudo nmap -sV 10.129.2.28
sudo nmap -sV -p 22,80,443 10.129.2.28
sudo nmap -p- -sV 10.129.2.28              # all ports + version detection
```

### -sC
**default script scan.** runs nmap's default set of NSE scripts against the target. equivalent to `--script default`.

```bash
sudo nmap -sC 10.129.2.28
sudo nmap -sC -sV -p 80 10.129.2.28        # scripts + version detection (common combo)
```

---

## host discovery

### -sn
**skip port scanning.** only does host discovery — figures out which hosts are alive without scanning any ports. used to be called `-sP` (ping scan).

```bash
sudo nmap -sn 10.129.2.0/24                # discover all hosts on the subnet
sudo nmap -sn 10.129.2.18                   # check if a single host is alive
```

### -Pn
**skip host discovery.** treats all hosts as alive and goes straight to scanning. useful when you know the host is up but ICMP is blocked.

```bash
sudo nmap -Pn 10.129.2.28                   # skip ping, just scan
sudo nmap -Pn -p 445 10.129.2.28            # scan a specific port without pinging first
```

### -PE
**ICMP echo request.** sends a ping to check if the host is alive. the classic "are you there?" packet.

```bash
sudo nmap -PE -sn 10.129.2.18               # ping scan using ICMP echo
```

### -PU
**UDP ping.** sends a UDP packet to discover hosts.

```bash
sudo nmap -PU -sn 10.129.2.0/24             # discover hosts using UDP ping
sudo nmap -PU53 -sn 10.129.2.0/24           # UDP ping on port 53 specifically
```

### -PA
**TCP ACK ping.** sends a TCP ACK packet for host discovery.

```bash
sudo nmap -PA -sn 10.129.2.0/24             # discover hosts using ACK ping
sudo nmap -PA80,443 -sn 10.129.2.0/24       # ACK ping on ports 80 and 443
```

### -PS
**TCP SYN ping.** sends a TCP SYN packet for host discovery.

```bash
sudo nmap -PS -sn 10.129.2.0/24             # discover hosts using SYN ping
sudo nmap -PS22,80,443 -sn 10.129.2.0/24    # SYN ping on specific ports
```

---

## port specification

### -p \<ports\>
**specify ports to scan.** you can define them a bunch of ways:

```bash
nmap -p 22 10.129.2.28                       # single port
nmap -p 22,80,443 10.129.2.28                # multiple ports
nmap -p 22-445 10.129.2.28                   # port range
nmap -p- 10.129.2.28                         # ALL 65,535 ports
nmap -p U:53,T:80 10.129.2.28                # UDP 53 and TCP 80
```

### -F
**fast scan.** only scans the top 100 most common ports instead of the default 1000.

```bash
sudo nmap -F 10.129.2.28
sudo nmap -F -sV 10.129.2.28                # fast scan with version detection
```

### --top-ports=\<n\>
**scan top N ports.** scans the N most commonly used ports based on nmap's frequency database.

```bash
sudo nmap --top-ports=10 10.129.2.28         # top 10 ports
sudo nmap --top-ports=100 10.129.2.28        # same as -F
sudo nmap --top-ports=1000 10.129.2.28       # same as default
```

---

## output

### -oN \<file\>
**normal output.** saves results in human-readable format (`.nmap`).

```bash
sudo nmap 10.129.2.28 -oN scan.nmap
```

### -oG \<file\>
**grepable output.** saves results in a format that's easy to parse with grep, awk, and cut (`.gnmap`).

```bash
sudo nmap 10.129.2.28 -oG scan.gnmap
```

### -oX \<file\>
**XML output.** saves results as structured XML data (`.xml`). can be converted to HTML with `xsltproc`.

```bash
sudo nmap 10.129.2.28 -oX scan.xml
xsltproc scan.xml -o scan.html              # convert to pretty HTML report
```

### -oA \<basename\>
**all formats.** saves results in normal, grepable, *and* XML at once. the move. just always use this.

```bash
sudo nmap 10.129.2.28 -oA target             # creates target.nmap, target.gnmap, target.xml
sudo nmap 10.129.2.28 -p- -sV -oA full_scan  # full scan, all outputs
```

---

## service and OS detection

### -A
**aggressive scan.** combines service detection (`-sV`), OS detection (`-O`), traceroute (`--traceroute`), and default scripts (`-sC`). one flag to rule them all.

```bash
sudo nmap -A 10.129.2.28
sudo nmap -A -p 80 10.129.2.28               # aggressive scan on just port 80
```

### -O
**OS detection.** tries to determine the target's operating system by analyzing TCP/IP fingerprints.

```bash
sudo nmap -O 10.129.2.28
sudo nmap -O --osscan-guess 10.129.2.28      # with aggressive guessing
```

---

## verbosity and debugging

### -v
**verbose.** shows more detailed output during the scan, including open ports as they're discovered in real time. use `-vv` for even more detail.

```bash
sudo nmap -v -p- -sV 10.129.2.28            # verbose full port scan
sudo nmap -vv -sS 10.129.2.28               # extra verbose SYN scan
```

### --reason
**show reasoning.** tells you *why* nmap classified a port or host the way it did (e.g., `syn-ack`, `arp-response`, `no-response`).

```bash
sudo nmap --reason -p 22,80 10.129.2.28
sudo nmap -sn --reason 10.129.2.18           # why does nmap think this host is up?
```

### --packet-trace
**show all packets.** displays every packet sent and received during the scan. great for understanding exactly what nmap is doing under the hood.

```bash
sudo nmap --packet-trace -p 21 10.129.2.28
sudo nmap --packet-trace -sn -PE 10.129.2.18 # trace a ping scan
```

### --stats-every=\<time\>
**periodic status updates.** shows scan progress at the specified interval (e.g., `--stats-every=5s` or `--stats-every=1m`).

```bash
sudo nmap -p- --stats-every=5s 10.129.2.28   # progress every 5 seconds
sudo nmap -p- --stats-every=1m 10.129.2.28   # progress every minute
```

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

```bash
sudo nmap -T4 -p- 10.129.2.28               # aggressive timing, all ports
sudo nmap -T1 -sS 10.129.2.28               # sneaky SYN scan for IDS evasion
```

### --min-rate \<number\>
**minimum packet rate.** tells nmap to send at least this many packets per second.

```bash
sudo nmap --min-rate 300 -p- 10.129.2.28     # at least 300 packets/sec
sudo nmap --min-rate 1000 -F 10.129.2.0/24   # fast sweep of a whole subnet
```

### --max-retries \<number\>
**max retry attempts.** how many times nmap will resend a probe to an unresponsive port (default: 10).

```bash
sudo nmap --max-retries 2 -p- 10.129.2.28    # only retry twice
sudo nmap --max-retries 0 -F 10.129.2.0/24   # no retries at all (fastest, least accurate)
```

### --initial-rtt-timeout \<time\>
**initial round-trip timeout.** how long nmap waits for the first response from a port.

```bash
sudo nmap --initial-rtt-timeout 50ms -F 10.129.2.0/24
```

### --max-rtt-timeout \<time\>
**max round-trip timeout.** the absolute maximum time nmap will wait for a response.

```bash
sudo nmap --max-rtt-timeout 100ms -F 10.129.2.0/24
sudo nmap --initial-rtt-timeout 50ms --max-rtt-timeout 100ms -F 10.129.2.0/24  # both together
```

---

## firewall/IDS evasion

### -D \<decoys\>
**decoy scan.** generates fake source IPs to hide your real one. use `RND:5` for 5 random decoys, or specify IPs manually. the decoy IPs should be alive to avoid triggering SYN-flood protections.

```bash
sudo nmap -D RND:5 -sS -p 80 10.129.2.28                      # 5 random decoys
sudo nmap -D 10.10.14.3,10.10.14.4,ME -p 80 10.129.2.28       # specific decoys, ME = your real IP
```

### -S \<IP\>
**spoof source IP.** makes the scan appear to come from a different IP address. useful for testing firewall rules from different subnets.

```bash
sudo nmap -S 10.129.2.200 -e tun0 -Pn -p 445 10.129.2.28      # always needs -e with it
```

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

```bash
sudo nmap -S 10.129.2.200 -e tun0 10.129.2.28    # spoof source IP over VPN
sudo nmap -e eth0 -sS 10.0.0.0/24                 # scan local network over ethernet
```

### --source-port \<port\>
**spoof source port.** makes the scan appear to come from a specific port. port 53 (DNS) is a classic choice because firewalls often trust DNS traffic.

```bash
sudo nmap --source-port 53 -sS -p 50000 10.129.2.28    # pretend to be DNS traffic
sudo nmap --source-port 80 -sS 10.129.2.28              # pretend to be HTTP traffic
```

### --disable-arp-ping
**disable ARP.** forces nmap to use ICMP instead of ARP for host discovery on local networks.

```bash
sudo nmap --disable-arp-ping -sn -PE 10.129.2.18                  # force ICMP ping
sudo nmap --disable-arp-ping -Pn -n --packet-trace -p 80 10.129.2.28  # clean packet trace
```

---

## scripting (NSE)

### --script \<scripts\>
**run specific scripts.** can be a script name, category, or comma-separated list:

```bash
sudo nmap --script banner -p 25 10.129.2.28                 # single script
sudo nmap --script vuln -p 80 -sV 10.129.2.28               # whole category
sudo nmap --script banner,smtp-commands -p 25 10.129.2.28   # multiple scripts
sudo nmap --script "http-*" -p 80 10.129.2.28               # wildcard — all http scripts
```

---

## DNS

### -n
**disable DNS resolution.** skips reverse DNS lookups. speeds things up and reduces noise.

```bash
sudo nmap -n 10.129.2.28                     # no DNS lookups
sudo nmap -n -Pn -sS 10.129.2.28             # common combo: no DNS, no ping, SYN scan
```

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

  **how to find it:** scan for port 53 on the target network. if something's listening, that's probably a DNS server.

  ```bash
  # find DNS servers on the network
  sudo nmap -p 53 10.129.2.0/24 -sS --open

  # check if it's actually DNS (not just port 53 open for something else)
  sudo nmap -p 53 -sV 10.129.2.1
  sudo nmap -p 53 -sU -sV 10.129.2.1         # DNS often runs on UDP 53 too

  # try a quick DNS query against it to confirm
  nslookup example.com 10.129.2.1
  dig @10.129.2.1 example.com
  ```

  you can also check nmap's service scan output — if you've already run `-sV` on the network, look for anything labeled `domain` or `dns` in the results. another trick: if you find an active directory domain controller (usually ports 88, 389, 445 open), it's almost always running DNS on port 53 too.
- **the network's default DNS** — whatever's in `/etc/resolv.conf` or `ipconfig /all`. these are usually the ones the network trusts.
- **public DNS** (8.8.8.8, 1.1.1.1) — useful as a fallback, but won't resolve internal hostnames.

**why this matters:** in a pentest, the company's internal DNS servers often resolve hostnames that reveal internal infrastructure — subdomains, internal services, dev servers. public DNS won't show you any of that. using `--dns-server` with an internal DNS server can uncover hosts you'd otherwise never find.

```bash
sudo nmap --dns-server 10.129.2.1 10.129.2.0/24               # use target's DNS server
sudo nmap --dns-server 10.129.2.1,8.8.8.8 10.129.2.0/24       # internal + fallback
```

---

## misc

### -b \<FTP relay host\>
**FTP bounce scan.** uses an FTP server to scan another host indirectly. old-school technique.

```bash
sudo nmap -b ftp.vulnerable.com 10.129.2.28
```

### --scanflags \<flags\>
**custom TCP flags.** lets you craft your own TCP flag combinations for custom scan types.

```bash
sudo nmap --scanflags SYNFIN -p 80 10.129.2.28    # SYN + FIN (weird combo, might confuse firewalls)
sudo nmap --scanflags URG -p 80 10.129.2.28        # just URG flag
```

### -iL \<file\>
**input from list.** reads target hosts from a file instead of specifying them on the command line.

```bash
sudo nmap -iL hosts.lst -sn                        # ping scan all hosts in the file
sudo nmap -iL targets.txt -sV -p- -oA full_scan    # full version scan from a list
```
