---
name: mailkite
description: Set up and operate MailKite email end-to-end — create an account, add a sending/receiving domain, set its DNS at the user's provider (Cloudflare, GoDaddy, Namecheap, Route 53, …), register a webhook, design templates, send mail, and confirm inbound delivery. Use whenever the user wants to send email, receive email as a webhook, give an AI agent its own inbox, wire up a domain for email, or debug MailKite delivery. Works via the MCP server, the `mailkite` CLI, any language SDK, or raw REST.
---

# MailKite

MailKite is a developer email platform: **receive email as a webhook**, **send with one API**, across unlimited domains — no mail server. This skill lets you drive the whole lifecycle for a user.

There are **four equivalent ways** to call MailKite. They all hit the same API and the same contract (`sdks/spec`). Pick per situation — see [reference/access-methods.md](reference/access-methods.md):

| Method | Use it when |
| --- | --- |
| **MCP tools** (`mailkite_*`) | You're an MCP client and the MailKite server is connected. Most ergonomic. |
| **`mailkite` CLI** | A terminal is available. Best for scripted, multi-step setup. `npx @mailkite/cli` |
| **SDK** (node/python/go/php/java/ruby) | You're editing the user's app and want native code. |
| **Raw REST** | Anywhere else — plain JSON + `Authorization: Bearer <token>`. |

Base URL `https://api.mailkite.dev`. **One bearer token** authenticates everything (sending *and* management). Today that token is the account JWT from login/signup (dedicated `mk_live_…` keys are on the roadmap).

> **Always prefer the CLI or MCP for actions over hand-rolled curl** unless the user is in an app where SDK code is the deliverable. The CLI is fully non-interactive with flags (`--json` for machine output), so it's the most reliable path for an agent.

## The end-to-end flow

This is the canonical path. Each step links to the tool/command; depth is in the reference files.

### 1. Account
```bash
npx @mailkite/cli signup --email "$EMAIL" --password "$PASSWORD" --json   # or: login
```
Stores the bearer token in `~/.mailkite/config.json`. MCP/SDK equivalent: POST `/api/auth/signup` (or `/login`) → `{ token, user }`. Save `token` and use it as the bearer everywhere.

### 2. Add a domain → get DNS records
```bash
npx @mailkite/cli domains add mail.theirapp.com --json
```
Returns the domain + the **DNS records** to publish (MX, SPF, DKIM, DMARC). MCP: `mailkite_create_domain`. Keep the returned `domain.id`.

### 3. Set the DNS at the user's provider
This is the step that needs their **third-party DNS host**. Identify the provider, then apply the records. Full per-provider playbooks (API, CLI, and MCP options for Cloudflare, GoDaddy, Namecheap, Route 53, and a manual fallback) are in **[reference/dns-providers.md](reference/dns-providers.md)** — read it before touching DNS.

The four records (from step 2) are always: `MX → mx.mailkite.dev` (priority 10), `TXT SPF`, `TXT DKIM` at `mailkite._domainkey.<domain>`, `TXT DMARC` at `_dmarc.<domain>`.

### 4. Verify
```bash
npx @mailkite/cli domains verify <domainId> --json
```
Checks live DNS over DoH; `status: "verified"` once MX resolves. DNS can take minutes to hours — poll, don't block. MCP: `mailkite_verify_domain`.

### 5. Register the inbound webhook
```bash
npx @mailkite/cli webhook set <domainId> https://theirapp.com/hooks/mailkite --json
```
Every email to that domain is parsed to JSON and POSTed there. MCP: `mailkite_set_webhook`. Get the signing secret and verify signatures — see **[reference/webhooks.md](reference/webhooks.md)**.

### 6. Design a template (optional)
Build the HTML. You can author raw HTML, or use the platform's block model. See **[reference/templates.md](reference/templates.md)** for the canonical template shape, a ready-to-edit starter, and the `/api/templates` CRUD endpoints.

### 7. Send a test email
```bash
npx @mailkite/cli send --from "hello@mail.theirapp.com" --to "$DEST" \
  --subject "MailKite test ✅" --html "<p>It works.</p>" --json
```
MCP: `mailkite_send`. `from` must be on a **verified** domain. Returns `{ id, status }`.

### 8. Confirm receipt (round-trip)
To prove inbound works, send to an address on the verified domain, then poll stored messages until it lands (no public tunnel needed):
```bash
npx @mailkite/cli messages tail --once --subject "MailKite test" --timeout 120 --json
```
MCP: `mailkite_list_messages` in a short poll loop. Then `mailkite_get_message` / `messages get <id>` for the full parsed message + attachments. Retry a failed webhook delivery with `deliveries retry <id>`.

### One-shot wizard
For an interactive or fully-scripted run of steps 1–7:
```bash
npx @mailkite/cli init --email "$EMAIL" --password "$PASSWORD" \
  --domain mail.theirapp.com --provider cloudflare \
  --webhook https://theirapp.com/hooks/mailkite --to "$DEST" --verify --json
```

## Everything the agent can do (capability map)

- **Account:** signup, login, whoami (`/api/auth/*`)
- **Domains:** add, list, get, verify, remove (`mailkite_*_domain`, `domains …`)
- **DNS:** read records, apply at provider, verify → [reference/dns-providers.md](reference/dns-providers.md)
- **Webhooks:** set, remove, test, get/rotate secret, **verify signatures locally** (`mailkite_verify_webhook` / `verify-webhook`) → [reference/webhooks.md](reference/webhooks.md)
- **Routes:** list, create (match → action → destination) for per-address handling
- **Templates:** author + CRUD, send a design test → [reference/templates.md](reference/templates.md)
- **Send:** transactional/test mail via `POST /v1/send`
- **Receive:** poll/list/get messages, attachments, retry deliveries
- **Access:** MCP, CLI, 6 SDKs, raw REST → [reference/access-methods.md](reference/access-methods.md) and [reference/api.md](reference/api.md)

## Gotchas

- **`from` must be on a verified domain** or the send is rejected. Verify (step 4) first.
- **Webhook signatures**: the HMAC is over the *raw* request bytes (`${t}.${rawBody}`). Verify before re-serializing — a parsed-and-restringified body won't match. Use `mailkite_verify_webhook` / `mailkite verify-webhook`.
- **DNS propagation is async.** `verify` may report `pending` for a while; that's expected, keep polling.
- **One token for everything.** The same bearer is used for `send` and management.
- **Full send fields work:** `cc`, `bcc`, `replyTo`, `inReplyTo` (sets In-Reply-To/References so replies thread), and `attachments` (`{filename,url}` is fetched; `{filename,content}` is base64) all go through. `from/to/subject` + one of `html`/`text` are required.

## Reference files
- [reference/access-methods.md](reference/access-methods.md) — MCP vs CLI vs SDK vs REST, install/config for each
- [reference/dns-providers.md](reference/dns-providers.md) — apply DNS on Cloudflare, GoDaddy, Namecheap, Route 53, manual
- [reference/webhooks.md](reference/webhooks.md) — payload shape, signature verification, receiving options
- [reference/templates.md](reference/templates.md) — email template model + starter + `/api/templates`
- [reference/api.md](reference/api.md) — every endpoint, auth, request/response shapes
