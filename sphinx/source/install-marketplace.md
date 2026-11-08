# Install Marketplace

The **Memchentic** plugin teaches your harness when to recall, when to
capture, and which tools fit which jobs. It ships in this repository so hosts
can import `https://github.com/memchentic/memchentic` directly.

The plugin does not replace the server. Connect MCP separately:
{doc}`connect-mcp`.

## Cursor

1. Dashboard → Settings → Plugins.
2. Import `https://github.com/memchentic/memchentic`.
3. Install **Memchentic**.

Or:

```shell
agent plugin marketplace add https://github.com/memchentic/memchentic.git
```

Cursor wires rules, skills, and commands.

## Claude Code

```shell
claude plugin marketplace add memchentic/memchentic
claude plugin install memchentic@memchentic
```

Skills + commands (Claude Code does not apply Cursor-style always-on rules —
load the `memchentic` index skill at session start).

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

1. Connect the MCP ({doc}`connect-mcp`).
2. Without a live MCP connection, the plugin's recall/capture gates stay idle.
3. Concepts and layout: {doc}`marketplace`. Tool list: {doc}`mcp-tools`.
