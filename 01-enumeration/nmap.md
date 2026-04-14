# nmap

> network mapper — the bread and butter of network scanning.

---

## what is it?

nmap is an open-source network analysis and security auditing tool (written in C, C++, Python, and Lua). it scans networks, finds hosts, identifies services and their versions, detects operating systems, and can even check if firewalls and IDS are configured properly.

it is *the* tool for network recon. if you're doing cybersecurity, you're using nmap.

## what's it used for?

- network security auditing
- simulating penetration tests
- testing firewall and IDS configurations
- network mapping
- identifying open ports
- service and version detection
- vulnerability assessment

## nmap's scanning techniques

nmap breaks down into five main categories:

1. **host discovery** — who's out there?
2. **port scanning** — what ports are open?
3. **service enumeration and detection** — what's running on those ports?
4. **OS detection** — what operating system is the target running?
5. **nmap scripting engine (NSE)** — scriptable interaction with target services (this is where it gets fun)

---

## syntax

```bash
nmap <scan types> <options> <target>
```

simple as that.

## scan types

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

---

## the TCP-SYN scan (-sS)

this is the default and probably the most popular scan method. here's why it's cool:

- it sends a packet with the **SYN flag** set
- it *never completes the three-way handshake*, so no full TCP connection is established
- this makes it fast — we're talking several thousand ports per second

### how to read the results

| target responds with | meaning |
|---|---|
| **SYN-ACK** | port is *open* |
| **RST** | port is *closed* |
| **nothing** | port is *filtered* (firewall probably dropped or ignored the packet) |

### example

```bash
sudo nmap -sS localhost
```

```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-11 22:50 UTC
Nmap scan report for localhost (127.0.0.1)
Host is up (0.000010s latency).
Not shown: 996 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
5432/tcp open  postgresql
5901/tcp open  vnc-1

Nmap done: 1 IP address (1 host up) scanned in 0.18 seconds
```

the output is straightforward — port number, state (open/closed/filtered), and the service name. four open ports here: SSH, HTTP, PostgreSQL, and VNC. each one of those is a potential way in.
