# DNS zone transfers

> when a DNS server accidentally hands you *everything*. the holy grail of subdomain discovery.

---

## what is a zone transfer?

a zone transfer is a mechanism for copying *all* DNS records in a zone (a domain and its subdomains) from one name server to another. it's meant for keeping backup DNS servers in sync with the primary one. totally legitimate, totally necessary.

the problem? if the server isn't properly configured, *anyone* can request a zone transfer. and that means you get the complete map — every subdomain, every IP, every record. no brute-forcing needed.

## how it works

1. **AXFR request** — the requesting server sends a zone transfer request (AXFR = full zone transfer) to the primary DNS server
2. **SOA record** — the primary server responds with its Start of Authority record (contains the zone's serial number so the requester can check if it's up to date)
3. **all the records** — the primary server sends every single DNS record in the zone, one by one. A, AAAA, MX, CNAME, NS, all of it
4. **transfer complete** — the primary server signals it's done
5. **ACK** — the requester confirms it got everything

---

## the vulnerability

in the early internet, it was common to let *any* client request a zone transfer. made administration easy, but it was a massive security hole. anyone — including attackers — could just ask for a complete copy of the zone file.

what an unauthorized zone transfer reveals:

- **every subdomain** — including hidden ones not linked from the main site (dev servers, staging environments, admin panels)
- **IP addresses** — for every subdomain, giving you targets for further recon
- **name server records** — hosting provider, potential misconfigurations
- **mail servers, TXT records, SRV records** — the whole picture

most modern DNS servers are configured to only allow zone transfers to trusted secondary servers. but misconfigurations still happen. always worth trying.

---

## how to do it

use `dig` with the `axfr` query type:

```bash
dig axfr @<nameserver> <domain>
```

### example — zonetransfer.me

(this is a site specifically set up to demonstrate the risk — safe to practice on)

```bash
dig axfr @nsztm1.digi.ninja zonetransfer.me
```

```
zonetransfer.me.        7200  IN  SOA   nsztm1.digi.ninja. robin.digi.ninja. 2019100801 ...
zonetransfer.me.        300   IN  HINFO "Casio fx-700G" "Windows XP"
zonetransfer.me.        301   IN  TXT   "google-site-verification=tyP28J7JAUHA9fw2..."
zonetransfer.me.        7200  IN  MX    0 ASPMX.L.GOOGLE.COM.
zonetransfer.me.        7200  IN  A     5.196.105.14
zonetransfer.me.        7200  IN  NS    nsztm1.digi.ninja.
zonetransfer.me.        7200  IN  NS    nsztm2.digi.ninja.
_sip._tcp.zonetransfer.me.    14000 IN SRV 0 0 5060 www.zonetransfer.me.
asfdbbox.zonetransfer.me.     7200  IN  A   127.0.0.1
canberra-office.zonetransfer.me. 7200 IN A  202.14.81.230
...
;; XFR size: 50 records (messages 1, bytes 2085)
```

50 records. the entire zone, handed over on a silver platter. subdomains like `canberra-office`, internal records pointing to `127.0.0.1`, SRV records revealing services — all of it just sitting there.

### trying it on a real target

first, find the target's name servers:

```bash
dig NS example.com +short
```

then try a zone transfer against each one:

```bash
dig axfr @ns1.example.com example.com
```

if it works, you just saved yourself hours of brute-forcing. if it doesn't (and it usually doesn't), you'll get a `Transfer failed` message. no harm done — move on to brute-forcing.

---

## key takeaways

- zone transfers are the fastest way to discover *all* subdomains — if the server allows it
- most servers are locked down now, but misconfigurations still happen
- always try it before resorting to brute-force. it's quick and costs nothing
- even a failed attempt can reveal info about the DNS server's configuration
- `dig axfr @<nameserver> <domain>` is all you need
