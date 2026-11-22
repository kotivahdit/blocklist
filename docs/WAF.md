# Using the Kotivahdit blocklist in a WAF or reverse proxy

The list is a plain IPv4 file, regenerated hourly:

```
https://raw.githubusercontent.com/kotivahdit/blocklist/main/ipv4-high-7d.txt
```

## Before you start

* **Use `ipv4-high-7d.txt` for blocking.** Use `ipv4-all-7d.txt` only for logging or challenge mode, because it may contain aggressive but harmless scanners.
* **Start in a soft mode**: count, log or challenge for a few days, then switch to block.
* **Refresh at most once per hour.** The list does not change more often than that.
* **Fail safe:** if a download fails or returns an unusually short file, keep the previous list. The scripts below do this.
* Lines starting with `#` are comments. All scripts below strip them.
* Shared, NAT and cloud addresses get re-assigned. Entries expire after 7 days of silence. To request removal, see [REMOVAL.md](../REMOVAL.md).

## Common download step (Linux)

Every recipe below starts with this. It writes the clean IP list to `/var/lib/kotivahdit/ips.txt` only when the download looks sane.

```bash
#!/bin/sh
# /usr/local/sbin/kotivahdit-fetch
set -eu
URL=https://raw.githubusercontent.com/kotivahdit/blocklist/main/ipv4-high-7d.txt
DIR=/var/lib/kotivahdit; mkdir -p "$DIR"
curl -fsS --max-time 60 "$URL" | grep -Ev '^\s*(#|$)' > "$DIR/ips.new"
# sanity check: at least 100 lines, each line an IPv4 address
[ "$(wc -l < "$DIR/ips.new")" -ge 100 ] || { echo "list too short, keeping old"; exit 1; }
grep -Evq '^[0-9]{1,3}(\.[0-9]{1,3}){3}$' "$DIR/ips.new" && { echo "unexpected content, keeping old"; exit 1; }
mv "$DIR/ips.new" "$DIR/ips.txt"
```

Run it hourly, for example `17 * * * * root /usr/local/sbin/kotivahdit-fetch && <reload step below>` in `/etc/cron.d/kotivahdit`.

## nginx

Uses the `geo` module (built into nginx), which stays fast with thousands of entries.

```bash
# reload step
awk '{print $1" 1;"}' /var/lib/kotivahdit/ips.txt > /etc/nginx/kotivahdit.conf.new \
 && mv /etc/nginx/kotivahdit.conf.new /etc/nginx/kotivahdit.conf \
 && nginx -t -q && nginx -s reload
```

```nginx
# http { } block
geo $kotivahdit_block {
    default 0;
    include /etc/nginx/kotivahdit.conf;
}

# server { } block
if ($kotivahdit_block) { return 403; }
```

Soft start: instead of `return 403`, log only, for example by adding `$kotivahdit_block` to your `log_format`.

## ModSecurity (v2 and v3, e.g. with the OWASP Core Rule Set)

```apache
SecRule REMOTE_ADDR "@ipMatchFromFile /var/lib/kotivahdit/ips.txt" \
    "id:1009001,phase:1,deny,status:403,log,msg:'Kotivahdit blocklist'"
```

Soft start: replace `deny,status:403` with `pass`. Reload the web server after each refresh so the file is read again.

## HAProxy

```haproxy
frontend web
    http-request deny if { src -f /var/lib/kotivahdit/ips.txt }
```

Reload step: `systemctl reload haproxy`.

## Cloudflare (IP List + custom rule)

1. **Manage Account → Configurations → Lists → Create list**, type *IP*, name `kotivahdit`. Check your plan's limits for list size.
2. **Security → WAF → Custom rules → Create rule**, expression `(ip.src in $kotivahdit)`, action **Managed Challenge** to start, **Block** later.
3. Refresh hourly through the API. `PUT` replaces all list items:

```bash
# needs an API token with "Account Filter Lists: Edit"
jq -R '{ip: .}' /var/lib/kotivahdit/ips.txt | jq -s . > /tmp/kv.json
curl -fsS -X PUT "https://api.cloudflare.com/client/v4/accounts/$ACCOUNT_ID/rules/lists/$LIST_ID/items" \
  -H "Authorization: Bearer $CF_TOKEN" -H "Content-Type: application/json" --data @/tmp/kv.json
```

## AWS WAF (v2 IP set)

1. Create an IPv4 IP set `kotivahdit` (scope `REGIONAL`, or `CLOUDFRONT` in us-east-1).
2. Add a rule to your web ACL: *IP set* `kotivahdit`, action **Count** to start, **Block** later.
3. Refresh hourly. AWS needs CIDR notation and a lock token:

```bash
TOKEN=$(aws wafv2 get-ip-set --name kotivahdit --scope REGIONAL --id "$IPSET_ID" --query LockToken --output text)
aws wafv2 update-ip-set --name kotivahdit --scope REGIONAL --id "$IPSET_ID" --lock-token "$TOKEN" \
  --addresses $(sed 's#$#/32#' /var/lib/kotivahdit/ips.txt)
```

## Firewalls with external feeds (no script needed)

Point the device at the raw URL and set the refresh to hourly:

* **Palo Alto:** Objects → External Dynamic Lists → type *IP List*.
* **FortiGate:** Security Fabric → External Connectors → Threat Feeds → *IP Address*.
* **pfSense (pfBlockerNG) / OPNsense (Aliases → URL Table):** add the raw URL.

## What our web trap sees (to test your WAF rules)

Your WAF's own rules should already stop the most common probes our web honeypot sees each week, for example requests for `/.env` and `/.git/config`, Hikvision `/SDK/webLanguage` exploitation, and Boa/GPON router logins. Live figures are on https://kotivahdit.github.io.

---
Questions or problems: open an issue, or email kotivahdit@proton.me. License: data [CC BY 4.0](../LICENSE).
