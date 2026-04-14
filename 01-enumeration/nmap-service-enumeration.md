# nmap — service enumeration

> knowing a port is open is step one. knowing *what's running on it* is where it gets interesting.

---

## why this matters

an exact version number lets you search for specific exploits. "ssh is open" is helpful. "OpenSSH 7.6p1 Ubuntu 4ubuntu0.3" is *powerful*. that level of detail lets you find precise vulnerabilities and even dig into source code for that version.

---

## version detection (-sV)

the recommended approach: do a quick port scan first (less traffic, less chance of getting caught), then come back with `-sV` on the ports you found.

```bash
sudo nmap 10.129.2.28 -p- -sV
```

```
PORT      STATE    SERVICE      VERSION
22/tcp    open     ssh          OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
25/tcp    open     smtp         Postfix smtpd
80/tcp    open     http         Apache httpd 2.4.29 ((Ubuntu))
110/tcp   open     pop3         Dovecot pop3d
139/tcp   filtered netbios-ssn
143/tcp   open     imap         Dovecot imapd (Ubuntu)
445/tcp   filtered microsoft-ds
993/tcp   open     ssl/imap     Dovecot imapd (Ubuntu)
995/tcp   open     ssl/pop3     Dovecot pop3d
```

look at that — we know exactly what software and versions are running. that's a *goldmine*.

## monitoring long scans

full port scans take a while. a few tricks to stay sane:

| method | how |
|---|---|
| press **spacebar** | shows current scan progress |
| `--stats-every=5s` | auto-update progress every 5 seconds (or `5m` for minutes) |
| `-v` or `-vv` | verbose mode — shows open ports as they're found in real time |

```bash
# progress updates every 5 seconds
sudo nmap 10.129.2.28 -p- -sV --stats-every=5s

# verbose — see ports as they're discovered
sudo nmap 10.129.2.28 -p- -sV -v
```

---

## banner grabbing — the details nmap misses

how does `-sV` actually work? mostly through **banners**. after connecting to a port, many services send a banner identifying themselves. nmap grabs that and reports it.

but here's the thing — nmap sometimes misses information buried in those banners. check this out:

```bash
sudo nmap 10.129.2.28 -p- -sV -Pn -n --disable-arp-ping --packet-trace
```

in the packet trace output you might see:

```
Callback: READ SUCCESS for EID 18 [10.129.2.28:25] (35 bytes): 220 inlane ESMTP Postfix (Ubuntu)..
```

nmap reported "Postfix smtpd" but the banner actually said **"Postfix (Ubuntu)"** — telling us the linux distro. nmap just didn't surface it. details like this matter.

### doing it manually with netcat

sometimes you gotta go old school:

```bash
nc -nv 10.129.2.28 25
```

```
Connection to 10.129.2.28 port 25 [tcp/*] succeeded!
220 inlane ESMTP Postfix (Ubuntu)
```

there it is — the full banner, nothing hidden.

### watching the handshake with tcpdump

if you really want to see what's happening at the network level:

```bash
# terminal 1 — capture traffic
sudo tcpdump -i eth0 host 10.10.14.2 and 10.129.2.28

# terminal 2 — connect to the service
nc -nv 10.129.2.28 25
```

the tcpdump output shows the full TCP handshake:

1. **[SYN]** — we send the initial connection request
2. **[SYN-ACK]** — target confirms and agrees
3. **[ACK]** — we acknowledge, connection established
4. **[PSH-ACK]** — target *pushes* data to us (the banner!) and acknowledges
5. **[ACK]** — we confirm we got the data

that `[PSH-ACK]` packet is where the banner lives. the PSH flag means "hey, i'm sending you data right now."

---

## key takeaway

nmap's `-sV` is great, but it's automated and can miss things. always consider manually grabbing banners with `nc` for critical services. the extra detail could be the difference between finding a way in and staying stuck.
