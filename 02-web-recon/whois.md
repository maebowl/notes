# WHOIS

> basically a phonebook for the internet. who owns what, and how to find out.

---

## what is it?

WHOIS is a query protocol that lets you look up information about registered internet resources — mostly domain names, but also IP address blocks and autonomous systems. you ask "who owns this domain?" and WHOIS tells you.

## what's in a WHOIS record?

| field | what it tells you |
|---|---|
| **domain name** | the domain itself (e.g., example.com) |
| **registrar** | the company where the domain was registered (GoDaddy, Namecheap, etc.) |
| **registrant contact** | person or org that registered it |
| **administrative contact** | person responsible for managing it |
| **technical contact** | person handling technical issues |
| **creation/expiration dates** | when it was registered, when it expires |
| **name servers** | servers that translate the domain to an IP address |

---

## why it matters for recon

WHOIS data is a goldmine during a pentest:

- **identifying people** — names, emails, phone numbers of domain managers. useful for social engineering and phishing
- **discovering infrastructure** — name servers and IPs reveal the target's network setup and potential misconfigurations
- **historical analysis** — services like WhoisFreaks let you see past WHOIS records, tracking changes in ownership, contacts, and infrastructure over time

## real-world scenarios

### phishing investigation

suspicious email comes in claiming to be from the company's bank. you WHOIS the domain in the link and find:
- registered *days ago*
- registrant info hidden behind a privacy service
- name servers on a known bulletproof hosting provider

red flags everywhere. that's almost certainly a phishing domain.

### malware C2 analysis

malware is phoning home to a remote server. WHOIS the C2 domain and you might find:
- registered with a free anonymous email
- registrant in a high-cybercrime country
- registrar known for lax abuse policies

now you know where to report it and can start tracking the infrastructure.

### threat intelligence

tracking a threat actor group? WHOIS their known domains and look for patterns:
- domains registered in clusters right before attacks
- shared name servers across campaigns (common infrastructure)
- recurring aliases or fake identities
- takedown history showing previous law enforcement action

these patterns become indicators of compromise (IOCs) that other organizations can use to detect future attacks.

---

## using WHOIS

### install it

```bash
sudo apt update
sudo apt install whois -y
```

### basic lookup

```bash
whois facebook.com
```

### reading the output

here's what a WHOIS lookup on facebook.com looks like (trimmed):

```
Domain Name: FACEBOOK.COM
Registry Domain ID: 2320948_DOMAIN_COM-VRSN
Registrar WHOIS Server: whois.registrarsafe.com
Updated Date: 2024-04-24T19:06:12Z
Creation Date: 1997-03-29T05:00:00Z
Registry Expiry Date: 2033-03-30T04:00:00Z
Registrar: RegistrarSafe, LLC

Domain Status: clientDeleteProhibited
Domain Status: clientTransferProhibited
Domain Status: clientUpdateProhibited
Domain Status: serverDeleteProhibited
Domain Status: serverTransferProhibited
Domain Status: serverUpdateProhibited

Name Server: A.NS.FACEBOOK.COM
Name Server: B.NS.FACEBOOK.COM
Name Server: C.NS.FACEBOOK.COM
Name Server: D.NS.FACEBOOK.COM

Registrant Name: Domain Admin
Registrant Organization: Meta Platforms, Inc.
```

### what we can pull from this

**domain registration:**
- registered in 1997, doesn't expire until 2033 — this is a well-established, legitimate domain
- registrar is RegistrarSafe, LLC

**ownership:**
- owned by Meta Platforms, Inc. (no surprise there)
- contact is just "Domain Admin" — large orgs often use generic contacts

**security posture:**
- all those `*Prohibited` statuses mean the domain is locked down tight — can't be deleted, transferred, or modified without explicit authorization on both client and server side
- this is standard practice for high-value domains

**infrastructure:**
- all four name servers are under `facebook.com` itself — Meta runs their own DNS infrastructure
- common for large orgs to maintain control over their own DNS resolution

### the limitations

WHOIS alone won't get you everything. it might not reveal individual employees or specific vulnerabilities. it's one piece of the puzzle — you need to combine it with other recon techniques (DNS enumeration, search engine dorking, social media analysis) to build the full picture.
