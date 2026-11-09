# Install Marketplace

The **Memchentic** plugin teaches your coding agent when to recall, when to
save, and how to use Memchentic well. Import it from
`https://github.com/memchentic/memchentic`.

Install the plugin and connect Memchentic. The agent then recalls and captures
on its own. You do not need to load a skill or run `/memchentic` to start that
habit.

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

Cursor injects the recall and capture loop through always-on plugin rules.

## Claude Code

```shell
claude plugin marketplace add memchentic/memchentic
claude plugin install memchentic@memchentic
```

Claude Code injects the same loop through the plugin session-start hook after
install. No manual skill load.

## GitHub Copilot CLI

```shell
copilot plugin marketplace add memchentic/memchentic
copilot plugin install memchentic@memchentic
```

Copilot CLI injects the loop through the plugin `sessionStart` hook after
install.

## OpenAI Codex

```shell
codex plugin marketplace add memchentic/memchentic
```

Codex injects the loop through the plugin `SessionStart` hook. One-time host
setup: enable `[features] hooks = true` and trust this plugin’s hook (via
`/hooks` or Settings → Hooks). After that, install is enough — no per-session
skill load.

## After install

1. Connect Memchentic ({doc}`connect-mcp`).
2. If the server is not connected, the plugin stays idle.
3. More about what the plugin teaches: {doc}`marketplace`.
   What the agent can do: {doc}`mcp-tools`.
