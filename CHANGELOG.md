# Changelog

* **2026-11-21** – v1.0: METHODOLOGY, REMOVAL, PRIVACY, CORRECTIONS published; Live page wording changed from
  "attacks" to "events" with definitions; README caveat corrected (we no longer claim "no false positives").
  Counting fixed: T-Pot passive fingerprinting (p0f/Suricata/fatt) and UDP/ICMP no longer counted as events, listed or
  reported – this reduced the T-Pot event count by ~88 %. Telnet-flood reports now require a completed TCP connection.
* **2026-11-21** – blocklist started (hourly, `high` / `all`, 7-day rolling expiry).
