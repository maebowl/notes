# nmap

> network mapper — the bread and butter of network scanning.

---

## what is it?

nmap is an open-source network analysis and security auditing tool. it scans networks, finds hosts, identifies services and their versions, detects operating systems, and can even check if firewalls and IDS are configured properly.

it is *the* tool for network recon. if you're doing cybersecurity, you're using nmap.

## syntax

```bash
nmap <scan types> <options> <target>
```

simple as that.

---

## the deep dives

- **[host discovery](nmap-host-discovery.md)** — finding which hosts are alive on a network. ARP vs ICMP, scanning ranges, lists, and single targets.

- **[port scanning](nmap-port-scanning.md)** — all 6 port states, SYN vs connect scans, how filtered ports work (dropped vs rejected), and UDP scanning.

- **[service enumeration](nmap-service-enumeration.md)** — version detection with `-sV`, banner grabbing, and why manual `nc` sometimes catches what nmap misses.

- **[saving results](nmap-saving-results.md)** — output formats (normal, grepable, XML), and turning scans into pretty HTML reports.

- **[scripting engine (NSE)](nmap-scripting-engine.md)** — lua scripts for recon, vuln scanning, brute-forcing, and the aggressive scan (`-A`).

- **[performance tuning](nmap-performance.md)** — timeouts, retries, packet rates, and timing templates (`-T 0` through `-T 5`).

- **[firewall and IDS/IPS evasion](nmap-firewall-evasion.md)** — ACK scans, decoys, source IP spoofing, DNS proxying, and staying under the radar.

---

## scan types at a glance

```
-sS/sT/sA/sW/sM    TCP SYN/Connect()/ACK/Window/Maimon scans
-sU                  UDP scan
-sN/sF/sX           TCP Null, FIN, and Xmas scans
--scanflags <flags>  customize TCP scan flags
-sI <zombie host>    idle scan
-sY/sZ              SCTP INIT/COOKIE-ECHO scans
-sO                  IP protocol scan
-b <FTP relay host>  FTP bounce scan
```

## nmap's five main capabilities

1. **host discovery** — who's out there?
2. **port scanning** — what ports are open?
3. **service enumeration and detection** — what's running on those ports?
4. **OS detection** — what operating system is the target running?
5. **nmap scripting engine (NSE)** — scriptable interaction with target services (this is where it gets fun)
