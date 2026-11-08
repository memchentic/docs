# Install Marketplace

The **Memchentic** plugin teaches your coding agent when to recall, when to
save, and how to use Memchentic well. Import it from
`https://github.com/memchentic/memchentic`.

The plugin does not replace the server. Connect Memchentic separately:
{doc}`connect-mcp`.

## Cursor

1. Dashboard → Settings → Plugins.
2. Import `https://github.com/memchentic/memchentic`.
3. Install **Memchentic**.

Or from a terminal:

```shell
agent plugin marketplace add https://github.com/memchentic/memchentic.git
```

## Claude Code

```shell
claude plugin marketplace add memchentic/memchentic
claude plugin install memchentic@memchentic
```

On Claude Code, load the Memchentic skill at the start of a session so the
agent follows the same recall and capture habits.

## GitHub Copilot CLI

```shell
copilot plugin marketplace add memchentic/memchentic
copilot plugin install memchentic@memchentic
```

## OpenAI Codex

```shell
codex plugin marketplace add memchentic/memchentic
```

## After install

1. Connect Memchentic ({doc}`connect-mcp`).
2. If the server is not connected, the plugin stays idle.
3. More about what the plugin teaches: {doc}`marketplace`.
   What the agent can do: {doc}`mcp-tools`.
