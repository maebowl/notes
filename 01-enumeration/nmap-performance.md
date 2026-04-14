# nmap — performance tuning

> scanning a huge network? you're gonna want to know these.

---

## the tradeoff

faster scans = less traffic = less chance of getting caught. but push it too hard and you'll miss open ports or alive hosts. it's a balancing act.

---

## timeouts (RTT)

nmap starts with a default round-trip-time timeout of 100ms. you can tweak this:

```bash
# default — 256 hosts, top 100 ports
sudo nmap 10.129.2.0/24 -F
# result: 10 hosts up, 39.44 seconds

# optimized timeouts
sudo nmap 10.129.2.0/24 -F --initial-rtt-timeout 50ms --max-rtt-timeout 100ms
# result: 8 hosts up, 12.29 seconds
```

3x faster, but we missed 2 hosts. if you set `--initial-rtt-timeout` too low, slow-responding hosts get skipped. speed has a cost.

| option | what it does |
|---|---|
| `--initial-rtt-timeout` | starting timeout value |
| `--max-rtt-timeout` | maximum timeout before giving up |

---

## retries

by default nmap retries each port **10 times** if it doesn't get a response. you can reduce that:

```bash
# default retries
sudo nmap 10.129.2.0/24 -F | grep "/tcp" | wc -l
# found: 23 ports

# zero retries
sudo nmap 10.129.2.0/24 -F --max-retries 0 | grep "/tcp" | wc -l
# found: 21 ports
```

missed 2 ports. again — faster but less thorough.

---

## packet rate

if you know the network bandwidth (like in a white-box pentest where you're whitelisted), you can crank up the packet rate:

```bash
# default
sudo nmap 10.129.2.0/24 -F -oN tnet.default
# 29.83 seconds, 23 ports

# minimum rate of 300 packets/sec
sudo nmap 10.129.2.0/24 -F -oN tnet.minrate300 --min-rate 300
# 8.67 seconds, 23 ports
```

same results, 3.5x faster. `--min-rate` is great when you have the bandwidth for it.

---

## timing templates (-T)

don't want to fiddle with individual settings? nmap has six presets:

| template | name | use case |
|---|---|---|
| `-T 0` | paranoid | IDS evasion, *painfully* slow |
| `-T 1` | sneaky | IDS evasion, still very slow |
| `-T 2` | polite | uses less bandwidth, won't overwhelm the target |
| `-T 3` | normal | the default |
| `-T 4` | aggressive | assumes a fast, reliable network |
| `-T 5` | insane | assumes an incredibly fast network, sacrifices accuracy |

```bash
# default
sudo nmap 10.129.2.0/24 -F -oN tnet.default
# 32.44 seconds

# insane mode
sudo nmap 10.129.2.0/24 -F -oN tnet.T5 -T 5
# 18.07 seconds
```

both found 23 ports in this case, but `-T 5` can absolutely miss things on slower or less reliable networks. use with caution.

---

## the bottom line

> click any flag to see its full explanation in the [glossary](nmap-glossary.md).

| option | what it does |
|---|---|
| [`--initial-rtt-timeout`](nmap-glossary.md#--initial-rtt-timeout-time) | starting timeout |
| [`--max-rtt-timeout`](nmap-glossary.md#--max-rtt-timeout-time) | max timeout |
| [`--max-retries`](nmap-glossary.md#--max-retries-number) | how many times to retry each port |
| [`--min-rate`](nmap-glossary.md#--min-rate-number) | minimum packets per second |
| [`-T <0-5>`](nmap-glossary.md#-t-0-5) | timing template preset |

the general rule: start with defaults, speed up *only* when you understand the tradeoff. missing a single open port could mean missing your way in.
