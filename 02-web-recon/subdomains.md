# subdomains

> the hidden rooms of a website. the main domain is the front door — subdomains are all the side entrances.

---

## what are subdomains?

subdomains are extensions of a main domain, used to organize different parts of a website or service. like `blog.example.com`, `shop.example.com`, or `mail.example.com`. in DNS, they're usually represented by A records (or AAAA for IPv6) mapping to IP addresses, or CNAME records aliasing to other domains.

## why do they matter for recon?

subdomains often hide the *interesting* stuff — things that aren't linked from the main site:

- **dev/staging environments** — testing areas with relaxed security and potential vulns. sometimes they expose sensitive info or unfinished features
- **hidden login portals** — admin panels, internal tools, stuff that's not supposed to be public
- **legacy applications** — old, forgotten apps running outdated software with known vulns
- **sensitive information** — internal docs, config files, things that should never have been exposed

finding these is often where a pentest gets really interesting.

---

## subdomain enumeration

two approaches: active and passive. ideally, you use both.

### active enumeration

you're directly interacting with the target's DNS. more thorough, but more detectable.

**zone transfers** — if a DNS server is misconfigured, it might hand you a complete list of all subdomains. rarely works these days because admins have gotten better about locking this down, but always worth trying.

**brute-force enumeration** — systematically test a list of potential subdomain names against the target. this is the bread and butter approach.

how it works:
1. **pick a wordlist** — common names like `dev`, `staging`, `blog`, `mail`, `admin`, `test`
2. **iterate** — append each word to the domain (e.g., `dev.example.com`, `staging.example.com`)
3. **DNS lookup** — check if each one resolves to an IP address
4. **validate** — if it resolves, it's real. maybe try visiting it in a browser to see what's there

wordlist types:
- **general-purpose** — broad list of common subdomain names. good when you don't know the target's naming conventions
- **targeted** — focused on specific industries or tech stacks. more efficient, fewer false positives
- **custom** — build your own based on intel from other recon

### passive enumeration

no direct interaction with the target. stealthier, but might miss things.

**certificate transparency (CT) logs** — public repos of SSL/TLS certificates. certificates often list associated subdomains in their Subject Alternative Name (SAN) field. gold mine.

**search engine operators** — use `site:example.com` on Google or DuckDuckGo to find indexed subdomains.

**online databases** — tools and services that aggregate DNS data from multiple sources, letting you search without touching the target.

---

## brute-force tools

| tool | description |
|---|---|
| **dnsenum** | comprehensive DNS enum — brute-force, zone transfers, google scraping, reverse lookups |
| **fierce** | recursive subdomain discovery with wildcard detection |
| **dnsrecon** | combines multiple techniques, customizable output |
| **amass** | actively maintained, integrates with other tools and data sources |
| **assetfinder** | simple and fast, great for quick lightweight scans |
| **puredns** | powerful DNS brute-forcing with filtering |

---

## dnsenum

the swiss army knife of DNS enumeration. written in Perl. does *everything*:

- DNS record enumeration (A, AAAA, NS, MX, TXT)
- automatic zone transfer attempts
- subdomain brute-forcing with wordlists
- google scraping for additional subdomains
- reverse DNS lookups
- WHOIS lookups

### basic usage

```bash
dnsenum --enum inlanefreight.com -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -r
```

| flag | what it does |
|---|---|
| `--enum` | shortcut that enables a bunch of tuning options |
| `-f <wordlist>` | path to the wordlist for brute-forcing |
| `-r` | recursive — if it finds a subdomain, it tries to enumerate *its* subdomains too |

### example output

```
-----   inlanefreight.com   -----

Host's addresses:
__________________
inlanefreight.com.                       300      IN    A        134.209.24.248

Brute forcing with /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt:
_______________________________________________________________________________________
www.inlanefreight.com.                   300      IN    A        134.209.24.248
support.inlanefreight.com.               300      IN    A        134.209.24.248

done.
```

it found `www` and `support` subdomains, both resolving to the same IP. that tells us they're probably on the same server — could be virtual hosts.

### good wordlists to use

SecLists has solid ones:
- `subdomains-top1million-5000.txt` — quick scan
- `subdomains-top1million-20000.txt` — solid middle ground
- `subdomains-top1million-110000.txt` — thorough but slow

the bigger the list, the longer the scan, but the more you'll find.
