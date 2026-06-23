# MailKite for Claude Code

The official [MailKite](https://mailkite.dev) plugin for Claude Code. Connect your agent to
MailKite — inbound email → webhook, plus a send API — with **one command and a browser
sign-in**. No API key to copy, no local process to run.

## Install

```text
/plugin marketplace add mailkite/claude-code
/plugin install mailkite@mailkite
```

Then authorize the connection:

```text
/mcp
```

Pick **mailkite → Authenticate**; a browser opens, you sign in to MailKite, and the server
flips to `✓ Connected`. That's it — the tools below are now available to your agent.

## What you get

- **Hosted MCP server** (`https://mcp.mailkite.dev/mcp`) with **OAuth 2.1** (PKCE) — the
  plugin registers it for you; no key in your config.
- **17 tools**: send mail, list/create/verify domains, manage catch-all webhooks, create
  inbound routes, list/read messages, retry deliveries, and verify webhook signatures
  (locally, no network).
- **Slash commands**:
  - `/mailkite:send-test [to]` — send a test email from a verified domain.
  - `/mailkite:test-inbound [domain]` — fire a signed test event at a domain's webhook.
  - `/mailkite:debug-webhook [message-id]` — diagnose a failing delivery and propose a fix.
- **The MailKite skill** — playbook + reference the agent loads on demand.

## Other ways to connect

This plugin is the easiest path. You can also:

- **Add the remote server directly** (no plugin):
  ```sh
  claude mcp add --transport http mailkite https://mcp.mailkite.dev/mcp
  # then: /mcp → Authenticate
  ```
  Or skip OAuth with a static key (headless/CI):
  ```sh
  claude mcp add --transport http mailkite https://mcp.mailkite.dev/mcp \
    --header "Authorization: Bearer mk_live_…"
  ```
- **Run the local server** (`@mailkite/mcp`) when you want a static key, offline use, or a
  custom base URL. See <https://mailkite.dev/docs/ai-agents>.

## Links

- Docs: <https://mailkite.dev/docs/ai-agents>
- Dashboard / API keys: <https://app.mailkite.dev/api-keys>

MIT © MailKite
