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

## Agent skill (standalone)

The MailKite **agent skill** also ships on its own — a `SKILL.md` playbook plus reference files
that drive the whole email lifecycle (account → domain → DNS → webhook → send → confirm inbound).
Use it in Claude Code or claude.ai without the plugin.

**Easiest — let your agent install it.** Paste this to Claude:

> Download the MailKite agent skill from
> `https://github.com/mailkite/claude-code/releases/latest/download/mailkite.zip` and unzip it
> into `~/.claude/skills/` (user-wide) or `.claude/skills/` (this project), then load it.

**Or grab it yourself.** Direct release download:
<https://github.com/mailkite/claude-code/releases/latest/download/mailkite.zip>

```sh
# Claude Code — user-wide
curl -L https://github.com/mailkite/claude-code/releases/latest/download/mailkite.zip -o mailkite.zip
unzip mailkite.zip -d ~/.claude/skills/

# …or scope it to one project
unzip mailkite.zip -d .claude/skills/
```

For **claude.ai / Claude Desktop**: download the zip above, then **Settings → Capabilities →
Skills → Upload skill** and drop it in.

Source: [`skills/mailkite`](https://github.com/mailkite/claude-code/tree/main/skills/mailkite).

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
