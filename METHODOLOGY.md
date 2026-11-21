# Methodology

*Version 1.0 · 2026-11-21 · changes are recorded in [CHANGELOG.md](CHANGELOG.md)*

Kotivahdit is a small volunteer network of internet-facing decoys (honeypots and tarpits) run from free-tier cloud
machines in Stockholm (Oracle Cloud) and Iowa (Google Cloud). Nothing in this network offers a real service – **every
inbound connection is unsolicited by definition**. This document explains what we record, how we decide what to list or
report, and where our numbers come from.

## 1. Sensors

| Sensor | What it does | What we record |
|---|---|---|
| SSH/Telnet honeypot | accepts logins, emulates a shell | source IP, credentials tried, commands, download URLs |
| Tarpits (SSH, Telnet, HTTP) | holds connections open as long as possible | source IP, connection duration |
| Web trap | serves fake sensitive files (`.env`, `.git/config` …) containing **canary cloud keys** | requested paths, user agent, canary key use |
| Fake Docker API (port 2375) | answers like an exposed Docker daemon, never runs anything | API calls, requested images |
| T-Pot sensor | ~25 open-source honeypots (ICS, web, mail, etc.) | events as logged by each honeypot |
| LLM-backed SSH honeypot | language-model shell with a deterministic shim | commands (used for our fingerprinting research) |
| GreyNoise sensor | community sensor for GreyNoise | sessions, as classified by GreyNoise |

Outbound traffic from the honeypot hosts is **restricted by a default-deny firewall**, so a compromised decoy cannot be used against third parties. On the T-Pot sensor outbound traffic is currently **logged and monitored**; enforcement is being switched on.

## 2. What the numbers on the Live page mean

* **Honeypot events, 24 h** – sum of logged interactions with our decoys: TCP connections, login attempts, commands,
  HTTP requests and GreyNoise sessions. **Passive fingerprinting records (p0f, Suricata, fatt) and UDP/ICMP traffic are
  excluded.** *An event is not a "hack".* The large majority are automated scans and password guessing.
* **Unique source IPs, 24 h** – distinct IPv4 addresses that produced at least one event, counted per sensor and summed
  (an address seen by two sensors counts twice).
* **Bot time wasted** – total seconds connections were held in our tarpits, last 7 days.
* **Novelty** – share of our source IPs, for that day, that were **not** on the public lists ipsum, FireHOL level 1 and level 3, or
  blocklist.de at the time of the check. Indicates how much our data adds; it is not a quality score.
* **Key thieves** – hosts that downloaded a canary key from our web trap **and then used it** against the cloud API.
  This is high-confidence malicious behaviour.

## 3. Blocklist inclusion criteria

| List | Included when |
|---|---|
| `high` | the IP (a) attempted a login, command or malware download on the SSH/Telnet honeypot, (b) sent a web exploit attempt to the web trap, (c) used a canary key, or (d) flooded a tarpit. IPs that GreyNoise classifies as **benign** (known research scanners) are excluded. |
| `all` | `high` plus IPs with ≥ 10 active TCP honeypot events in 24 h on the T-Pot sensor (passive fingerprinting and UDP excluded). May include aggressive but non-exploiting scanners. |

* Only public IPv4 addresses; our own addresses and private ranges are never listed.
* **Expiry:** an IP is dropped after **7 days without new activity** (rolling). Addresses on shared, NAT or cloud
  infrastructure get re-assigned; long retention would punish the next user.
* We never list an address based on third-party data alone.

## 4. Outbound reporting

| Destination | What | Automation |
|---|---|---|
| AbuseIPDB | attacking IPs – only TCP sources that completed a connection (never UDP or SYN-only traffic, which can be spoofed), max. one report per IP per 24 h | automatic |
| abuse.ch (URLhaus, MalwareBazaar, ThreatFox) | malware download URLs and samples seen in the honeypot | automatic |
| SANS ISC / DShield, AlienVault OTX, CrowdSec, GreyNoise | sensor data feeds | automatic |
| Netcraft, hosting providers, CERTs | phishing sites, compromised hosts, credential thieves | **manual – a human reviews every report** |

Reports about websites or organisations are never sent automatically. See [CORRECTIONS.md](CORRECTIONS.md) for why.

## 5. Limits – please read

* A honeypot sees only what is sent to *it*. Our view is a small sample of internet background noise, biased by our
  locations and the services we emulate.
* IP geolocation tells you where an address is registered, **not who is behind it**. We do not attribute activity to
  countries or groups based on geolocation.
* We do not interact with attackers' systems (no "hack-back"), and we do not publish credentials, keys or personal data.

Questions or corrections: **kotivahdit@proton.me** · removal requests: [REMOVAL.md](REMOVAL.md)
