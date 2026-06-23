# Inbound webhooks & signatures

When mail arrives at any address on a verified domain, MailKite parses it and
`POST`s a JSON event to the domain's webhook (or a per-address route).

## The `email.received` payload

```json
{
  "id": "msg_2Hk9…",
  "type": "email.received",
  "from": { "address": "ada@example.com" },
  "to": [{ "address": "support@yourapp.mailkite.dev" }],
  "subject": "Re: invoice #1042",
  "text": "Looks good — approved!",
  "html": "<p>Looks good — approved!</p>",
  "threadId": "<a1b2c3@mail.example.com>",
  "auth": { "spf": "pass", "dkim": "pass", "dmarc": "pass", "spam": "ham" },
  "attachments": [
    { "id": "msg_2Hk9…:0", "filename": "po.pdf", "contentType": "application/pdf",
      "size": 18213, "url": "https://api.mailkite.dev/att/2Hk9…/0?exp=…&sig=…" }
  ]
}
```

Respond with any `2xx` to acknowledge. Keep the handler fast; do heavy work out
of band. Use `id` to make processing idempotent. Attachment `url`s are signed,
credential-free, and valid for 7 days.

## Verifying the signature

Every delivery carries `x-mailkite-signature: t=<ms>,v1=<hex>`.

1. Read the **raw, unparsed** request body (exact bytes).
2. Concatenate `` `${t}.` `` + raw body.
3. Compute `HMAC-SHA256(secret, that)` as lowercase hex.
4. Constant-time compare to `v1`.
5. Reject if `t` is more than ~5 minutes old (replay protection).

Get/rotate the signing secret: `GET` / `POST /api/webhooks/secret[/rotate]`.

**Use the SDK helper** rather than hand-rolling: every SDK ships
`verifyWebhook(signature, rawBody, secret)` (Ruby: `Mailkite.verify_webhook`,
MCP: `mailkite_verify_webhook`). It does the HMAC, constant-time compare, and
replay window in one call.

```js
import { MailKite } from "mailkite";
if (!MailKite.verifyWebhook(req.headers["x-mailkite-signature"], rawBody, SECRET)) {
  return res.sendStatus(401);
}
```

> Sign the **raw bytes**, not a parsed-and-re-serialized object — key order and
> whitespace change the bytes and break the signature.

## Routing

A catch-all sends every address on a domain to the domain webhook. To split
traffic, create routes (`POST /api/routes`):

```json
{ "match": "support@yourapp.mailkite.dev", "action": "webhook",
  "destination": "https://yourapp.com/hooks/support" }
```

`action`: `webhook` (destination = URL), `forward` (destination = address),
`store`, or `drop`. Every inbound message is stored regardless, so you can list,
inspect, and replay it.

## Testing & retries

- `POST /api/domains/:id/webhook/test` — fire a signed sample event.
- `POST /api/deliveries/:id/retry` — re-deliver the exact stored payload.
- No public tunnel? Poll `GET /api/messages` (CLI: `messages tail`) to confirm a
  round-trip locally.
