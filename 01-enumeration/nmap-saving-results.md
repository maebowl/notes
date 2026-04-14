# nmap — saving results

> save every scan. future you will thank present you.

---

## output formats

nmap can save in three formats:

| flag | format | file extension |
|---|---|---|
| `-oN` | normal (human-readable) | `.nmap` |
| `-oG` | grepable (easy to parse with grep/awk) | `.gnmap` |
| `-oX` | XML (structured data) | `.xml` |

or just save all three at once with `-oA`:

```bash
sudo nmap 10.129.2.28 -p- -oA target
```

this creates `target.nmap`, `target.gnmap`, and `target.xml` in your current directory.

---

## what each format looks like

### normal (-oN)

```
# Nmap 7.80 scan initiated Tue Jun 16 12:14:53 2020 as: nmap -p- -oA target 10.129.2.28
Nmap scan report for 10.129.2.28
Host is up (0.053s latency).
Not shown: 4 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
25/tcp open  smtp
80/tcp open  http
```

just what you'd see in the terminal. good for reading.

### grepable (-oG)

```
Host: 10.129.2.28 ()  Ports: 22/open/tcp//ssh///, 25/open/tcp//smtp///, 80/open/tcp//http///
```

everything on one line per host. perfect for piping into grep, awk, cut, etc.

### XML (-oX)

structured data. not fun to read raw, but *very* useful for one thing:

### turning XML into a pretty HTML report

```bash
xsltproc target.xml -o target.html
```

open that HTML file in a browser and you've got a clean, professional-looking report. great for documentation and sharing results with people who don't live in a terminal.

---

## the move

just always use `-oA`. it's one flag, gives you everything, and you never have to wonder "did i save that scan?" save yourself the headache.
