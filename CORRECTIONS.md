# Corrections log

We publish our mistakes. Each entry: what happened, impact, fix.

## 2026-11-21 – 8 legitimate websites reported as phishing to Netcraft

* **What happened:** our typosquatting monitor (checks look-alike domains of Finnish brands) automatically reported
  8 domains to Netcraft at 08:57 UTC. All 8 were legitimate businesses whose names merely resembled a brand.
* **Cause:** the rule matched a brand name *anywhere* on a page that had a login form. Short names such as "op" or
  "tori" matched ordinary words.
* **Impact:** reports were sent to Netcraft, which reviews every report before taking action. Netcraft support
  confirmed the same day that the reported URLs were **not blocked or flagged** – no harm to the sites.
* **Fix (same day):** retraction sent for all 8 URLs via Netcraft's mistake-report API; the rule now requires the full
  brand name in the page title; **automatic reporting of websites was disabled** – a human reviews every such report.

## 2026-11-21 – Inflated event counts and non-compliant T-Pot reports

* **What happened:** our T-Pot sensor counted *every* logged record as an event – including passive fingerprinting
  (p0f, Suricata, fatt) and UDP/ICMP traffic. About 88 % of its 24-hour "events" were not interactions with a decoy.
  The same query fed the `all` blocklist and our daily AbuseIPDB bulk report, which could include UDP sources
  (spoofable). Telnet-flood reports were based on SYN-packet counts.
* **Fix (same day):** only active TCP honeypot events are counted, listed or reported; flood reports require a
  completed TCP connection; the Live page now says "honeypot events" with a definition instead of "attacks".

## 2026-11-21 – Outbound filtering on the T-Pot sensor was not active

* **What happened:** during a review we found that the outbound logging rule on our T-Pot sensor had not survived a
  restart, so container egress was neither logged nor restricted, contrary to our methodology.
* **Impact:** none observed – T-Pot's decoys are low/medium interaction and do not execute attacker code.
* **Fix (same day):** persistent egress policy (DNS/NTP/HTTP(S) only, everything else logged and dropped, UDP replies
  rate-limited per destination), re-applied automatically every 10 minutes.
