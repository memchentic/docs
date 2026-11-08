# Connect MCP

Point your coding agent at Memchentic so it can save and recall memories.
Without a working connection, the optional marketplace plugin stays idle.

## Where to point it

Use the MCP URL for your Memchentic server:

| Setup | Typical URL |
| --- | --- |
| Default Compose ports | `http://your-host:18082/mcp` |
| Reverse proxy / one public site | `https://your-host/mcp` |

`your-host` is the machine or DNS name that reaches *your* Memchentic
server — not your laptop unless the server is actually running there.

## Choose how to sign in

| Method | When to use |
| --- | --- |
| Authorize (browser login) | Preferred when your harness has an **Authorize** / **Needs login** control (Cursor and similar) |
| Personal API key | Harnesses that only support a fixed Authorization header, or headless agents |
| Admin API key | Break-glass only |

## Authorize (browser login)

1. Add an HTTP MCP server with **only** the MCP URL — no Authorization header
   and no API key in the config.
2. In the harness, open that server and use **Authorize**, **Needs login**, or
   the equivalent (in Cursor: MCP settings → the server → authenticate /
   Authorize).
3. Confirm the client name looks right, then sign in with your Memchentic
   email and password and click **Authorize**.
4. When the browser returns to the harness, the server should show as
   connected and tools become available.

You need an account first. Create one or sign in through the Memchentic web
UI, then come back to Authorize. New accounts are not created on the
authorize page itself.

Example Cursor-style config (URL only):

```json
{
  "mcpServers": {
    "memchentic": {
      "url": "http://your-host:18082/mcp"
    }
  }
}
```

After Authorize succeeds, the harness keeps the session refreshed. You do not
paste a token into config.

If your site uses an external login provider (Google, Okta, and so on), the
same **Authorize** control starts that provider’s login instead of the built-in
email/password form. Operators must register the callback URL from
{doc}`environment-variables` on that provider.

## Personal API key

1. Sign in to the Memchentic web UI.
2. Open the account menu → **API Keys**.
3. **Generate key** (or **Cycle key**). Copy it immediately — it is shown once.
4. Point the harness at the MCP URL and send the key as a Bearer token.

```text
Authorization: Bearer <your-api-key>
```

```json
{
  "mcpServers": {
    "memchentic": {
      "url": "http://your-host:18082/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_API_KEY"
      }
    }
  }
}
```

The key acts as you: your personal space and every group you belong to. Prefer
a personal key over the admin API key for everyday agents.

Do not put both an API key and Authorize on the same server entry — pick one.

## By harness

Labels differ; the idea is the same.

**Cursor** — Settings → MCP → add the MCP URL. Prefer Authorize (URL only).
For a static key, add the Authorization header as above.

**Claude Code** — add Memchentic to project or user MCP config with the same
URL; Authorize when available, otherwise the Bearer header.

**GitHub Copilot / Codex** — register the same URL and credentials in that
product’s MCP settings.

If the host uses a different config key (for example `servers` instead of
`mcpServers`), keep the URL and auth values the same.

## Check it worked

- The harness shows Memchentic as connected / authenticated.
- The agent can use basic tools such as showing who you are or listing spaces.
- If you have the Memories page open, new saves from the agent appear there.

What the tools do: {doc}`mcp-tools`. Running the server: {doc}`docker` or
{doc}`helm`. Optional plugin for recall/capture habits:
{doc}`install-marketplace`.
