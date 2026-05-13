# DNS

> the internet's GPS. translates human-readable domain names into IP addresses so computers can actually find each other.

---

## what is DNS?

you type `www.example.com` into your browser, but your computer doesn't speak words — it speaks numbers (IP addresses). DNS is the translator that converts `www.example.com` into something like `192.0.2.1` so your computer knows where to go.

without DNS, you'd have to memorize IP addresses for every website. no thanks.

---

## how a DNS lookup works

think of it like a relay race. your request gets passed down a chain until someone has the answer:

1. **your computer checks its cache** — "have i been here before?" if yes, done. if not...
2. **DNS resolver (usually your ISP)** — checks *its* cache. if it doesn't know, it starts asking around
3. **root name server** — doesn't know the exact address, but knows who handles `.com`, `.org`, etc. points the resolver to the right TLD server
4. **TLD name server** — knows which authoritative server handles `example.com` specifically
5. **authoritative name server** — the final stop. *this* server actually has the IP address. sends it back
6. **resolver caches the answer** — saves it for next time, then hands it to your computer
7. **your computer connects** — now it knows where to go

the whole thing takes milliseconds.

---

## the hosts file

before DNS even gets involved, your computer checks a local file called the **hosts file**. it's a manual override — you can map any hostname to any IP address.

**where to find it:**
- windows: `C:\Windows\System32\drivers\etc\hosts`
- linux/mac: `/etc/hosts`

**format:**
```
<IP Address>    <Hostname>
```

**common uses:**

```
# local development
127.0.0.1       myapp.local

# testing connectivity
192.168.1.20    testserver.local

# blocking a website (redirect to nowhere)
0.0.0.0         unwanted-site.com
```

edit it with admin/root privileges. changes take effect immediately, no restart needed.

---

## DNS zones and zone files

a **zone** is a chunk of the domain namespace managed by a specific entity. like, `example.com` and all its subdomains (`mail.example.com`, `blog.example.com`) would typically be one zone.

the **zone file** is a text file on the DNS server that defines all the records for that zone. here's a simplified example:

```dns
$TTL 3600 ; Default Time-To-Live (1 hour)
@       IN SOA   ns1.example.com. admin.example.com. (
                2024060401 ; Serial number
                3600       ; Refresh interval
                900        ; Retry interval
                604800     ; Expire time
                86400 )    ; Minimum TTL

@       IN NS    ns1.example.com.
@       IN NS    ns2.example.com.
@       IN MX 10 mail.example.com.
www     IN A     192.0.2.1
mail    IN A     198.51.100.1
ftp     IN CNAME www.example.com.
```

(the "IN" stands for "Internet" — it's the protocol class. you'll almost always see "IN" since it refers to standard internet protocols.)

---

## DNS record types

this is the important stuff. each record type stores different information:

| type | name | what it does | example |
|---|---|---|---|
| **A** | address record | maps hostname → IPv4 | `www.example.com. IN A 192.0.2.1` |
| **AAAA** | IPv6 address | maps hostname → IPv6 | `www.example.com. IN AAAA 2001:db8::7334` |
| **CNAME** | canonical name | alias pointing to another hostname | `blog.example.com. IN CNAME webserver.example.net.` |
| **MX** | mail exchange | specifies mail servers for the domain | `example.com. IN MX 10 mail.example.com.` |
| **NS** | name server | delegates zone to an authoritative name server | `example.com. IN NS ns1.example.com.` |
| **TXT** | text record | arbitrary text, often for verification or security policies | `example.com. IN TXT "v=spf1 mx -all"` |
| **SOA** | start of authority | admin info about the zone (primary NS, email, timers) | `example.com. IN SOA ns1.example.com. admin.example.com. ...` |
| **SRV** | service record | hostname and port for specific services | `_sip._udp.example.com. IN SRV 10 5 5060 sipserver.example.com.` |
| **PTR** | pointer record | reverse DNS — maps IP → hostname | `1.2.0.192.in-addr.arpa. IN PTR www.example.com.` |

---

## why DNS matters for recon

DNS is *packed* with information about a target's infrastructure:

- **uncovering assets** — subdomains, mail servers, name servers. a CNAME record pointing to `oldserver.example.net` might lead to something vulnerable
- **mapping infrastructure** — NS records reveal the hosting provider, A records for `loadbalancer.example.com` reveal load balancers, MX records show where email goes
- **monitoring changes** — a new `vpn.example.com` subdomain might mean a new entry point. a TXT record with `_1password=...` tells you they use 1Password (useful for social engineering)

---

## DNS tools

| tool | what it does | best for |
|---|---|---|
| **dig** | versatile DNS lookup, supports all record types, detailed output | manual queries, in-depth analysis, zone transfers |
| **nslookup** | simpler DNS lookup | quick checks, basic queries |
| **host** | streamlined lookup, concise output | fast A/AAAA/MX checks |
| **dnsenum** | automated enumeration, brute-forcing, zone transfers | discovering subdomains efficiently |
| **fierce** | recursive subdomain enumeration with wildcard detection | finding subdomains and potential targets |
| **dnsrecon** | combines multiple recon techniques, various output formats | comprehensive DNS enumeration |
| **theHarvester** | OSINT tool, gathers info from multiple sources | emails, employees, data associated with a domain |

---

## dig (domain information groper)

the go-to DNS tool. flexible, detailed, and customizable.

### common commands

| command | what it does |
|---|---|
| `dig example.com` | default A record lookup |
| `dig example.com A` | get IPv4 address |
| `dig example.com AAAA` | get IPv6 address |
| `dig example.com MX` | find mail servers |
| `dig example.com NS` | find authoritative name servers |
| `dig example.com TXT` | get TXT records |
| `dig example.com CNAME` | get CNAME records |
| `dig example.com SOA` | get start of authority record |
| `dig example.com ANY` | get all records (many servers ignore this) |
| `dig @1.1.1.1 example.com` | query a specific DNS server |
| `dig +trace example.com` | show the full resolution path |
| `dig -x 192.168.1.1` | reverse lookup (IP → hostname) |
| `dig +short example.com` | just the answer, nothing else |
| `dig +noall +answer example.com` | only the answer section |

### reading dig output

```bash
dig google.com
```

```
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 16449
;; flags: qr rd ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             0       IN      A       142.251.47.142

;; Query time: 0 msec
;; SERVER: 172.23.176.1#53(172.23.176.1) (UDP)
;; WHEN: Thu Jun 13 10:45:58 SAST 2024
;; MSG SIZE  rcvd: 54
```

breaking that down:

**header:**
- `NOERROR` — query was successful
- `qr` — this is a response (query response)
- `rd` — recursion was requested
- `ad` — resolver considers the data authentic

**question section:**
- "what's the A record for google.com?"

**answer section:**
- `google.com. 0 IN A 142.251.47.142` — the IP is `142.251.47.142`, TTL is 0 (cache immediately expires)

**footer:**
- query took 0ms, answered by server `172.23.176.1` over UDP

### just want the answer?

```bash
dig +short hackthebox.com
```

```
104.18.20.126
104.18.21.126
```

clean and simple.

> **heads up:** some servers detect and block excessive DNS queries. respect rate limits, and always get permission before doing heavy DNS recon on a target.
