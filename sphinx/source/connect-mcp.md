# Connect MCP

Point your agent harness at the Memchentic MCP endpoint and authenticate.
Without a live connection, marketplace recall/capture craft stays idle.

Principals come from the connection credentials. The server never trusts a
client-supplied user id.

## Endpoint

The MCP mounts at `/mcp` on the application listener.

| Setup | Typical URL |
| --- | --- |
| Repo Compose defaults | `http://your-host:18082/mcp` |
| Reverse proxy / single port | `https://your-host/mcp` |

`your-host` is whatever machine or DNS name reaches the running server — not
loopback on your laptop unless the server is actually local to that machine.

## Choose how to authenticate

| Method | When to use |
| --- | --- |
| MCP Authorize (OAuth) | Preferred when the harness has an **Authorize** / **Needs login** control (Cursor and similar) |
| Per-user API key | Harnesses that only support a static `Authorization` header, or headless agents |
| `ADMIN_API_KEY` | Break-glass only — same principal as `ADMIN_EMAIL` |

## Authorize (OAuth button)

Use this when the harness can open a browser login for the MCP server.
With the default `OAUTH_PROVIDER=internal`, Memchentic shows its own authorize
page (email + password, then **Authorize**).

1. Add an HTTP MCP server with only the `/mcp` URL — **no**
   `Authorization` header and **no** API key in the config.
2. In the harness MCP UI, open the server and use **Authorize**,
   **Needs login**, or the equivalent control (Cursor: MCP settings → the
   server entry → authenticate / Authorize).
3. The harness registers itself with Memchentic, then opens a browser page
   titled **memchentic Authorize**. Confirm the client name and redirect host
   match the app you started from.
4. Sign in with your Memchentic email and password, then click **Authorize**.
5. The browser returns to the harness. The connection should show as connected
   / authenticated; tools become available to the agent.

You need an account first. Register or sign in via the GUI, then return to
Authorize. New users cannot create an account on the authorize page itself.

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

After Authorize succeeds, the harness stores and refreshes the token. You do
not paste a JWT into config.

When `OAUTH_PROVIDER` is an external IdP, the same harness **Authorize**
control still starts the flow; browser login is that IdP instead of the
internal authorize form. Register
`{BASE_URL}/api/auth/oauth/callback` on the IdP for GUI Continue-with; MCP
clients use dynamic registration against Memchentic’s OAuth endpoints.

## API key

Use a per-user key when the harness cannot run Authorize, or when you want a
long-lived Bearer secret in config.

1. Sign in to the Memchentic GUI.
2. Open the account menu → **API Keys**.
3. **Generate key** (or **Cycle key** to replace an existing one). Copy the
   secret immediately — it is shown once.
4. Point the harness at `/mcp` and send the key as a Bearer token.

```text
Authorization: Bearer <your-api-key>
```

Example config many hosts accept:

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

The key acts as you: personal space plus every group you belong to. Admins can
cycle keys for directory users from the same page. Prefer a per-user key over
`ADMIN_API_KEY` for day-to-day agents.

Do not put both an API key header and OAuth Authorize on the same server
entry — pick one method.

## Configure by harness

Exact labels differ; the shape is the same.

**Cursor** — Settings → MCP → add a server with the `/mcp` URL. Prefer
Authorize (URL only), then use the server’s authenticate / Authorize control.
For a static key, add the `Authorization` header as above.

**Claude Code** — add Memchentic to the project or user MCP config with the
same URL; use Authorize when the host supports it, otherwise the Bearer
header.

**GitHub Copilot / Codex** — register an MCP server against the same URL and
credentials in that harness’s MCP settings.

If the host uses a different schema key (for example `servers` instead of
`mcpServers`), keep the URL and auth values the same.

## Check it worked

- The harness shows the Memchentic server as connected / authenticated.
- The agent can call tools such as `me` or `space_list`.
- GUI Memories updates when the agent writes (if you have the page open).

Tool reference: {doc}`mcp-tools`. Run the server: {doc}`docker` or
{doc}`helm`. Teach the harness the memory loop: {doc}`install-marketplace`.
