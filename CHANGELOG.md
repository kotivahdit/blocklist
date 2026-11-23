# Changelog

* **2026-11-23** – [LEGITIMATE-INTEREST.md](LEGITIMATE-INTEREST.md): GDPR Art. 6(1)(f) assessment (purpose, necessity, balancing) following EDPB Guidelines 1/2024 v2.0.

* **2026-11-23** – IPs are now published 24 h after first sighting, and first/last-seen dates are rounded to the day (CSV columns renamed `first_seen_date_utc`, `last_seen_date_utc`). This protects sensor addresses from timing correlation; blocking value is unchanged for repeat offenders.

* **2026-11-22** – [docs/WAF.md](docs/WAF.md): recipes for nginx, ModSecurity, HAProxy, Cloudflare, AWS WAF and firewall feeds, with a fail-safe download script.

* **2026-11-21** – MISP feed (`misp/`) and STIX 2.1 bundle (`stix2-high-7d.json`) added for the `high` list.

* **2026-11-21** – T-Pot sensor: enforced egress policy + UDP reply rate limit; raw log retention ≤ 30 days enforced on the
  SSH/web honeypot host (daily cleanup).

* **2026-11-21** – v1.0: METHODOLOGY, REMOVAL, PRIVACY, CORRECTIONS published; Live page wording changed from
  "attacks" to "events" with definitions; README caveat corrected (we no longer claim "no false positives").
  Counting fixed: T-Pot passive fingerprinting (p0f/Suricata/fatt) and UDP/ICMP no longer counted as events, listed or
  reported – this reduced the T-Pot event count by ~88 %. Telnet-flood reports now require a completed TCP connection.
* **2026-11-21** – blocklist started (hourly, `high` / `all`, 7-day rolling expiry).
