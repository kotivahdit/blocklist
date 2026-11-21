# Kotivahdit blocklist

Hourly IPv4 blocklist from the **Kotivahdit** volunteer honeypot network (Finland; sensors in Stockholm and Iowa).
🔴 Live numbers: https://kotivahdit.github.io · Toolkit: https://github.com/kotivahdit/toolkit

**Every IP here attacked our own sensors.** IPs expire automatically after **7 days of silence**.

| File | Contents |
|---|---|
| `ipv4-high-7d.txt` | **High confidence**: login / command / malware-download attempts on our SSH-telnet honeypot, tarpit victims, web exploit attempts (GreyNoise-benign scanners removed), honeytoken key thieves, connection floods |
| `ipv4-all-7d.txt` | High + IPs with ≥ 10 events / 24 h on our T-Pot sensor (may include aggressive scanners) |
| `ipv4-high-24h.txt`, `ipv4-all-24h.txt` | Same, last 24 hours only |
| `key-thieves-7d.txt` | Hosts that **stole a bait cloud key from our web trap and tried to use it** |
| `ipv4-all-7d.csv` | ip, first/last seen, sources, confidence |

Raw URLs (stable): `https://raw.githubusercontent.com/kotivahdit/blocklist/main/ipv4-high-7d.txt`

## Machine-readable formats
| Format | URL |
|---|---|
| **MISP feed** (add as *Feed → MISP Feed*) | `https://raw.githubusercontent.com/kotivahdit/blocklist/main/misp/` |
| **STIX 2.1 bundle** (indicators, `high`, valid 7 days after last activity) | `https://raw.githubusercontent.com/kotivahdit/blocklist/main/stix2-high-7d.json` |
| CSV with first/last seen and sources | `https://raw.githubusercontent.com/kotivahdit/blocklist/main/ipv4-all-7d.csv` |

All formats are regenerated hourly. TLP:CLEAR.

## Use

**ipset / iptables**
```bash
ipset create kotivahdit hash:ip -exist
curl -s https://raw.githubusercontent.com/kotivahdit/blocklist/main/ipv4-high-7d.txt | grep -v '^#' | xargs -n1 ipset add kotivahdit -exist
iptables -I INPUT -m set --match-set kotivahdit src -j DROP
```
**nftables**
```bash
nft add set inet filter kotivahdit '{ type ipv4_addr; }'
curl -s https://raw.githubusercontent.com/kotivahdit/blocklist/main/ipv4-high-7d.txt | grep -v '^#' | sed 's/$/,/' | xargs nft add element inet filter kotivahdit { }
```
**Palo Alto (External Dynamic List)** – Objects → External Dynamic Lists → Type *IP List*, Source: the raw URL above, check hourly.

**CrowdSec / pfSense / OPNsense** – add the raw URL as a URL table / blocklist.

## Caveats
* Honeypot data is strong evidence, not proof: shared, NAT and cloud addresses get re-assigned, and an aggressive but harmless scanner can land on the `all` list. That is why entries expire after 7 days and why `high` and `all` are separate.
* Our own addresses and private ranges are never listed. Use at your own risk.

## Documentation
[Methodology](METHODOLOGY.md) · [Removal requests](REMOVAL.md) · [Corrections log](CORRECTIONS.md) · [Privacy](PRIVACY.md) · [Changelog](CHANGELOG.md)

License: data [CC BY 4.0](LICENSE) · Kotivahdit, kotivahdit@proton.me
