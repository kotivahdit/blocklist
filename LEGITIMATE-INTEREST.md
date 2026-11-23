# Legitimate interest assessment (GDPR Art. 6(1)(f))

*Kotivahdit blocklist and honeypot data. First version 2026-11-23. Reviewed whenever we change what we collect or publish.*

An IP address can be personal data under EU law. This page explains why we process and publish attacking IP addresses,
and how we keep that processing necessary and proportionate. It follows the three-part test used by European data
protection authorities (purpose, necessity, balancing), as described in the EDPB
[Guidelines 1/2024 on legitimate interest](https://www.edpb.europa.eu/system/files/2026-11/edpb_guidelines_202401_legitimateinterest_v2_en.pdf)
(v2.0, adopted 10 November 2026), which state that network and information security measures can in principle rely on
Art. 6(1)(f) when the necessity and balancing tests are met (see also [GDPR recital 49](https://gdpr-info.eu/recitals/no-49/)).

## 1. Purpose

Network and information security: detecting, measuring and reducing automated abuse (brute-force logins, malware
spreading, exploitation attempts, stolen-credential use) by letting others block sources that are attacking right now,
and by reporting abuse to providers who can stop it.

## 2. Necessity

* **Only attackers are recorded.** Our sensors host no real service and are not advertised to the public. A legitimate
  user has no reason to connect to them, so the data is about sources that sent unsolicited, mostly malicious traffic.
* **Only what is needed for blocking is published:** the IP address, the type of source (honeypot, tarpit, web trap…),
  and first/last-seen **dates**. We do not publish passwords that might be real, session contents, keys, or anything
  about uninvolved people.
* **Less intrusive alternatives considered:** publishing only aggregates would not let anyone block an attack; publishing
  raw logs would reveal far more than needed. A short-lived IP list is the least data that still achieves the purpose.

## 3. Balancing

| Risk to the person behind an address | How we limit it |
|---|---|
| The address is shared, NAT'd or re-assigned (an innocent user is listed) | Entries **expire automatically after 7 days** without new activity; `high` and `all` lists are separated; free **removal on request** ([REMOVAL.md](REMOVAL.md)) and a public [corrections log](CORRECTIONS.md) |
| A hijacked device (e.g. a home router) is listed | We publish *what* the address did, never *who* it belongs to; for Finnish consumer connections we also notify the operator's abuse contact so the device can be cleaned |
| Over-retention | Raw sensor logs ≤ 30 days, list entries ≤ 7 days after last activity (see [PRIVACY.md](PRIVACY.md)) |
| Precise timing could reveal patterns about a person | Since 2026-11-23 addresses are published **24 h after first sighting** and dates are at **day precision** |
| Reasonable expectations | Sending attack traffic to systems you do not control is commonly expected to be logged, reported and blocked |

**Reporting to authorities.** We do not run an automatic feed of all addresses to law enforcement. Individual incidents
or campaigns may be reported to the Finnish National Cyber Security Centre (NCSC-FI), in line with the EDPB's guidance
that systematic, generalised reporting is not covered by legitimate interest while reporting specific cases can be.

**Outcome:** we consider the processing necessary for a legitimate security purpose, and the safeguards above keep the
impact on individuals low. Objections and requests: kotivahdit@proton.me or [REMOVAL.md](REMOVAL.md).
