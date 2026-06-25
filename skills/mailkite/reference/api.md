# MailKite API reference

Base URL: `https://api.mailkite.dev`. All requests are JSON over HTTPS. Auth is a
single bearer token (`Authorization: Bearer <token>`) — today the account JWT
from signup/login is used for both sending and management.

## Auth (public)

| Method | Path | Body | Returns |
| --- | --- | --- | --- |
| POST | `/api/auth/signup` | `{ email, password }` | `{ token, user }` |
| POST | `/api/auth/login` | `{ email, password }` | `{ token, user }` |
| POST | `/api/auth/google` | `{ code }` | `{ token, user }` |

## Sending (bearer)

| Method | Path | Body | Returns |
| --- | --- | --- | --- |
| POST | `/v1/send` | send message (below) | `202 { id, status }` |

Send body: `from` (required, address on a verified domain), `to` (required,
string or array), `subject` (required), `html` and/or `text`, optional `cc`,
`bcc`, `replyTo`, `inReplyTo`, `attachments[]` (`{ filename, url }` or
`{ filename, content, contentType }`).

## Agents & routing (bearer)

| Method | Path | Body | Returns |
| --- | --- | --- | --- |
| POST | `/v1/agent` | agent message (below) | `200 { ok, text, messageId }` |
| POST | `/v1/route` | route message (below) | `202 { id, routed, action }` |

`/v1/agent` sends a message straight to one of your inbox agents and returns its
reply — separate from inbound routing (it never matches/overrides routes). Body:
`text` (required), optional `subject`, `from`, `html`, `routeId` (target a
specific agent route), `address` (target the agent whose route matches it), and
`model` (override the model). With no selector it uses the account's **default
agent** (its most recently created `action: "agent"` route).

`/v1/route` hands a message to one of your **registered** routes and runs that
route's action (agent, webhook). Arbitrary destinations are not allowed. Body:
one of `routeId` or `address` (required), `from` (required), optional `subject`,
`text`, `html`.

## Domains (bearer)

| Method | Path | Notes |
| --- | --- | --- |
| GET | `/api/domains` | List domains + webhook URLs. |
| POST | `/api/domains` | `{ domain }` → domain + DNS records. |
| GET | `/api/domains/:id` | One domain with DNS + webhook. |
| DELETE | `/api/domains/:id` | Remove a domain. |
| POST | `/api/domains/:id/verify` | Re-check DNS; updates `status`. |
| PUT | `/api/domains/:id/webhook` | `{ url }` — set catch-all webhook. |
| DELETE | `/api/domains/:id/webhook` | Remove the webhook. |
| POST | `/api/domains/:id/webhook/test` | Fire a signed test event. |

## Routes (bearer)

| Method | Path | Notes |
| --- | --- | --- |
| GET | `/api/routes` | List inbound routing rules. |
| POST | `/api/routes` | `{ match, action, destination }`. |

`action` is one of `webhook` · `forward` · `store` · `drop`.

## Messages & deliveries (bearer)

| Method | Path | Notes |
| --- | --- | --- |
| GET | `/api/messages` | List stored inbound messages. |
| GET | `/api/messages/:id` | Full message + deliveries + attachments. |
| POST | `/api/deliveries/:id/retry` | Re-deliver a stored message. |

## Webhook secret (bearer)

| Method | Path | Returns |
| --- | --- | --- |
| GET | `/api/webhooks/secret` | `{ secret }` (creates one if missing). |
| POST | `/api/webhooks/secret/rotate` | `{ secret }` (new secret). |

## Templates (bearer)

| Method | Path | Notes |
| --- | --- | --- |
| GET | `/api/templates` | List templates. |
| POST | `/api/templates` | `{ name, subject?, json?, html?, text?, theme? }` → `201` row. `name` required. |
| GET | `/api/templates/:id` | One template. |
| PUT | `/api/templates/:id` | Partial update (same fields). |
| DELETE | `/api/templates/:id` | `{ ok: true }`. |
| POST | `/api/templates/test` | `{ to, subject?, html?, text? }` — send a rendered test. |

## Attachments

**Inbound** attachments are served from signed, time-limited URLs returned in the
webhook + Messages API: `GET /att/:mid/:idx?exp=…&sig=…` — no credential needed.
Valid 7 days; expired → `410`, tampered → `403`.

**Outbound** — upload a file once and reference it by secure URL instead of
base64-inlining it on every send:

| POST | `/v1/attachments` | Store a file → `{ id, url, filename, contentType, size, expiresAt }`. |

The SDKs/CLI/MCP method `uploadAttachment` takes the file four ways — a local
`path` (read off disk → raw-binary upload, the easy path), raw `bytes`, a remote
`url` (we fetch & re-host it), or base64 `content`. Over raw HTTP it's an S3-style
PUT: the bytes are the body, with `Content-Type` + `?filename=&retentionDays=`
(retentionDays ∈ {7,30,90,365}, default 7). Pass the returned `url` to `send()` as
`{ filename, url }` or link it inline. `path` is local-only — not on the hosted MCP.

## Conventions & errors

- IDs are type-prefixed: `dom_`, `rte_`, `msg_`, `dlv_`, `usr_`, `tpl_`.
- Timestamps are Unix epoch **milliseconds**.
- Errors are `{ "error": "message" }` with standard status codes:
  `400` bad request · `401` unauthorized · `403` forbidden · `404` not found ·
  `409` conflict · `410` gone · `502` upstream send/delivery failed.

See also the canonical machine contract in [`sdks/spec/api.json`](../../../sdks/spec/api.json).
