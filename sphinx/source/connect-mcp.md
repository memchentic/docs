# Connect MCP

Point your agent harness at the Memchentic MCP endpoint and authenticate.
Without a live connection, marketplace recall/capture craft stays idle.

## Endpoint

The MCP mounts at `/mcp` on the application listener.

| Setup | Typical URL |
| --- | --- |
| Repo Compose defaults | `http://your-host:18082/mcp` |
| Reverse proxy / single port | `https://your-host/mcp` |

`your-host` is whatever machine or DNS name reaches the running server — not
loopback on your laptop unless the server is actually local to that machine.

## Authentication

Principals come from the connection credentials. The server never trusts a
client-supplied user id.

| Method | When to use |
| --- | --- |
| Per-user API key | Day-to-day agents acting as a normal account |
| Bearer JWT | Preferred when the harness supports MCP Authorize / OAuth |
| `ADMIN_API_KEY` | Break-glass only — same principal as `ADMIN_EMAIL` |

Create a per-user API key in the GUI after you sign in (Account → API keys).
Send it as:

```text
Authorization: Bearer <your-api-key-or-jwt>
```

## Configure the harness

Add an HTTP MCP server that targets `/mcp` and sends the Authorization
header. Exact UI labels differ by host; the shape is the same.

**Cursor** — Settings → MCP → add a server (or edit MCP config) with the URL
and header above.

**Claude Code** — add Memchentic to the project's or user's MCP server config
with the same URL and Authorization header.

**GitHub Copilot / Codex** — register an MCP server against the same URL and
credentials in that harness's MCP settings.

Example JSON many hosts accept:

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

Replace the URL and key for your deployment. If the host uses a different
schema key (for example `servers` instead of `mcpServers`), keep the URL and
header values the same.

Tool reference: {doc}`mcp-tools`. Run the server: {doc}`docker` or
{doc}`helm`. Teach the harness the memory loop: {doc}`install-marketplace`.
