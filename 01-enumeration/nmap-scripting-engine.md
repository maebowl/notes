# nmap — scripting engine (NSE)

> this is where nmap goes from scanner to full-on recon tool.

---

## what is NSE?

the nmap scripting engine lets you run scripts (written in Lua) that interact with services on your target. want to grab banners, brute-force logins, check for vulns, or enumerate users? there's probably a script for that.

## script categories

there are **14 categories**, and honestly some of these are wild:

| category | what it does |
|---|---|
| **auth** | checks for authentication credentials |
| **broadcast** | discovers hosts via broadcasting |
| **brute** | brute-forces logins against services |
| **default** | the scripts that run with `-sC` |
| **discovery** | evaluates accessible services |
| **dos** | checks for denial of service vulns (use carefully) |
| **exploit** | tries to exploit known vulnerabilities |
| **external** | uses external services for processing |
| **fuzzer** | sends weird/unexpected data to find vulns (slow) |
| **intrusive** | could negatively affect the target — be careful |
| **malware** | checks if the target is infected |
| **safe** | non-intrusive, non-destructive scripts |
| **version** | extends service detection |
| **vuln** | identifies specific vulnerabilities |

---

## how to use scripts

### run default scripts

```bash
sudo nmap <target> -sC
```

### run a whole category

```bash
sudo nmap <target> --script <category>
```

### run specific scripts

```bash
sudo nmap <target> --script <script-name>,<script-name>,...
```

### example — banner + smtp commands

```bash
sudo nmap 10.129.2.28 -p 25 --script banner,smtp-commands
```

```
PORT   STATE SERVICE
25/tcp open  smtp
|_banner: 220 inlane ESMTP Postfix (Ubuntu)
|_smtp-commands: inlane, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS,
                 ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8,
```

look at that — the banner tells us it's Ubuntu, and `smtp-commands` shows us what commands the SMTP server accepts. that `VRFY` command? that could help us enumerate *existing users* on the target. details matter.

---

## the aggressive scan (-A)

don't feel like specifying everything manually? `-A` combines:
- service detection (`-sV`)
- OS detection (`-O`)
- traceroute (`--traceroute`)
- default scripts (`-sC`)

```bash
sudo nmap 10.129.2.28 -p 80 -A
```

```
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-generator: WordPress 5.3.4
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: blog.inlanefreight.com
Aggressive OS guesses: Linux 2.6.32 (96%), Linux 3.2 - 4.9 (96%)
```

from one scan we now know: Apache 2.4.29, WordPress 5.3.4, the site title, and it's probably Linux. *that's* the power of `-A`.

---

## vulnerability scanning with NSE

this is where it gets really fun. the `vuln` category will actively probe for known vulnerabilities:

```bash
sudo nmap 10.129.2.28 -p 80 -sV --script vuln
```

```
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-enum:
|   /wp-login.php: Possible admin folder
|   /readme.html: Wordpress version: 2
|   /: WordPress version: 5.3.4
| http-wordpress-users:
|   Username found: admin
| vulners:
|   cpe:/a:apache:http_server:2.4.29:
|       CVE-2019-0211   7.2
|       CVE-2018-1312   6.8
|       CVE-2017-15715  6.8
```

it found the WordPress login page, enumerated usernames (found **admin**), and pulled up CVEs for the Apache version. that's a *lot* of attack surface from one command.

---

## key takeaways

- `-sC` for default scripts — should be in most of your scans
- `-A` for the lazy aggressive scan that does everything
- `--script vuln` when you want to actively hunt for vulnerabilities
- individual scripts give you surgical precision for specific services
- full list of scripts: https://nmap.org/nsedoc/index.html
