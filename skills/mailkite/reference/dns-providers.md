# Applying DNS at the user's provider

After `POST /api/domains` you get four records to publish. Names shown for
`mail.app.com` — substitute the real domain.

| Type | Name | Value | Purpose |
| --- | --- | --- | --- |
| MX | `mail.app.com` | `mx.mailkite.dev` (priority 10) | Receive |
| TXT | `mail.app.com` | `v=spf1 include:mailkite.dev ~all` | SPF |
| TXT | `mailkite._domainkey.mail.app.com` | `v=DKIM1; k=rsa; p=…` | DKIM |
| TXT | `_dmarc.mail.app.com` | `v=DMARC1; p=none;` | DMARC |

Always copy the **exact** values from the API/dashboard response (the DKIM key is
per-domain). Then `POST /api/domains/:id/verify`. MX resolving to
`mx.mailkite.dev` flips the domain to `verified`; SPF/DKIM/DMARC are monitored
separately. Propagation is async — poll, don't block.

## Cloudflare

- **API:** `POST https://api.cloudflare.com/client/v4/zones/{zone}/dns_records`
  with `Authorization: Bearer <CF_TOKEN>` and `{ "type","name","content","priority"?,"ttl":1 }`.
  Set **DNS-only** (not proxied) — proxying doesn't apply to MX/TXT anyway.
- **MCP/skill:** if a Cloudflare MCP/skill is connected, create the records there.
- One record per row above; MX needs `priority: 10`.

## GoDaddy

- **API:** `PATCH https://api.godaddy.com/v1/domains/{domain}/records` with
  `Authorization: sso-key {KEY}:{SECRET}` and an array of
  `{ "type","name","data","priority"?,"ttl":3600 }`. Use the **host** part for
  `name` (e.g. `mailkite._domainkey.mail` for a `mail.app.com` subdomain), not the FQDN.

## Namecheap

- **API:** `namecheap.domains.dns.setHosts` (it **replaces** all host records —
  fetch existing with `getHosts` first and merge). Or the dashboard: Advanced DNS
  → Add New Record. `Host` is the subpart (`@`, `mail`, `mailkite._domainkey.mail`).

## Route 53

- **CLI:** `aws route53 change-resource-record-sets --hosted-zone-id <id>
  --change-batch <json>` with `UPSERT` actions. MX value is `"10 mx.mailkite.dev"`
  in a single `ResourceRecords` entry. TXT values must be **double-quoted**
  (`"\"v=spf1 include:mailkite.dev ~all\""`).

## Manual fallback

If you can't reach an API, present the four records as a copy-paste table and ask
the user to add them at their registrar, then run `verify`. Re-run verify until
all four are green; this can take minutes to hours.
