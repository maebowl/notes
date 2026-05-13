# web reconnaissance

> the foundation of any real security assessment. you gotta know what you're looking at before you start poking it.

---

## what is it?

web recon is systematically collecting information about a target website or web application. it's the prep work — the "information gathering" phase of a pentest. you're building a map of everything before you ever try to exploit anything.

## the goals

- **identify assets** — find everything that's publicly accessible: web pages, subdomains, IP addresses, tech stack
- **discover hidden info** — backup files, config files, internal docs that shouldn't be exposed
- **analyze the attack surface** — what technologies, configurations, and entry points exist?
- **gather intelligence** — key personnel, email addresses, behavioral patterns for social engineering

attackers use this to tailor their attacks. defenders use it to find and fix things before attackers do. same process, different intent.

---

## active vs passive recon

two fundamentally different approaches, and the distinction matters.

### active recon — you're touching the target

you're directly interacting with the target system. more info, but more risk of getting caught.

| technique | what it does | tools | detection risk |
|---|---|---|---|
| **port scanning** | find open ports and services | nmap, masscan | high |
| **vulnerability scanning** | probe for known vulns | nessus, openvas, nikto | high |
| **network mapping** | map network topology | traceroute, nmap | medium-high |
| **banner grabbing** | grab service identification info | netcat, curl | low |
| **OS fingerprinting** | identify the operating system | nmap (`-O`) | low |
| **service enumeration** | determine service versions | nmap (`-sV`) | low |
| **web spidering** | crawl and map the website structure | burp suite, OWASP ZAP, scrapy | low-medium |

### passive recon — you're not touching anything

you're gathering info from publicly available sources. stealthier, but you only get what's already out there.

| technique | what it does | tools | detection risk |
|---|---|---|---|
| **search engine queries** | google dorking, finding exposed info | google, shodan | very low |
| **WHOIS lookups** | domain registration details | `whois` CLI, online tools | very low |
| **DNS analysis** | subdomains, mail servers, infrastructure | dig, nslookup, dnsenum | very low |
| **web archive analysis** | historical snapshots of the site | wayback machine | very low |
| **social media analysis** | employee info, org structure | linkedin, twitter | very low |
| **code repositories** | exposed creds, vulns in public repos | github, gitlab | very low |

---

## the deep dives

- **[WHOIS](whois.md)** — domain registration lookups, what they reveal, and why they matter
- **[DNS](dns.md)** — how DNS works, record types, zones, and using dig for recon

*more topics will be linked here as they're added.*
