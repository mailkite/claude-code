# Access methods: MCP · CLI · SDK · REST

All four hit the same API and the same contract (`sdks/spec`). One bearer token
authenticates everything.

## MCP tools (`mailkite_*`)

Best when you're an MCP client (Claude Desktop, Claude Code, Cursor) and the
MailKite server is connected.

**Preferred — hosted remote server + OAuth (no key to copy):**
`https://mcp.mailkite.dev/mcp`. In Claude Code, install the plugin:

```text
/plugin marketplace add mailkite/claude-code
/plugin install mailkite@mailkite
/mcp        # → mailkite → Authenticate (browser sign-in)
```

…or add the remote server directly:
`claude mcp add --transport http mailkite https://mcp.mailkite.dev/mcp` (then `/mcp` to auth),
or with a static key for headless use:
`--header "Authorization: Bearer mk_live_…"`.

**Local server** (static key, offline, custom base URL, or stdio-only clients):

```json
{
  "mcpServers": {
    "mailkite": {
      "command": "npx",
      "args": ["-y", "@mailkite/mcp"],
      "env": { "MAILKITE_API_KEY": "<token>" }
    }
  }
}
```

Both expose the same tools, mirroring the API one-to-one: `mailkite_send`,
`mailkite_create_domain`, `mailkite_verify_domain`, `mailkite_set_webhook`,
`mailkite_test_webhook`, `mailkite_list_messages`, `mailkite_get_message`,
`mailkite_retry_delivery`, `mailkite_verify_webhook`, … For the local server,
`MAILKITE_BASE_URL` overrides the endpoint.

## `mailkite` CLI

Best for scripted, multi-step setup in a terminal. Non-interactive with flags;
`--json` for machine output. No install needed via `npx`:

```bash
npx @mailkite/cli login --email "$EMAIL" --password "$PASSWORD" --json
npx @mailkite/cli domains add mail.app.com --json
npx @mailkite/cli send --from hello@mail.app.com --to you@example.com \
  --subject "Hi" --html "<p>It works.</p>" --json
```

The token is cached in `~/.mailkite/config.json`. `npx @mailkite/cli init …`
runs the whole signup→domain→DNS→webhook→send flow in one command.

## SDK (node · python · php · java · go · ruby)

Best when you're editing the user's app and want native code. Same shape in
every language — a low-level `request()` plus one method per endpoint.

```bash
npm install mailkite        # pip install mailkite-dev · composer require mailkite/mailkite
                            # gem install mailkite · go get github.com/mailkite/mailkite-go
```

```js
import { MailKite } from "mailkite";
const mk = new MailKite(process.env.MAILKITE_API_KEY);
await mk.send({ from, to, subject, html });
```

Methods: `send`, `listDomains`, `createDomain`, `getDomain`, `deleteDomain`,
`verifyDomain`, `setWebhook`, `deleteWebhook`, `testWebhook`, `listRoutes`,
`createRoute`, `listMessages`, `getMessage`, `retryDelivery`, plus the local
`verifyWebhook(signature, body, secret)` helper. (Go exports `PascalCase`.)

## Raw REST

Anywhere else — plain JSON + `Authorization: Bearer <token>`:

```bash
curl https://api.mailkite.dev/v1/send \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{ "from":"hello@mail.app.com","to":"you@example.com","subject":"Hi","text":"It works." }'
```

See [api.md](api.md) for every endpoint.

## Picking one

- Agent in an MCP client → **MCP**.
- Agent with a shell, doing setup → **CLI** (most reliable; fully flag-driven).
- Writing app code → **SDK** in that language.
- Everything else / quick checks → **REST**.
