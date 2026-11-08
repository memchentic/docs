# Memory Types

Memchentic stores memories in **spaces**. Every memory is one of two **kinds**:
an **entry** or a **guide**. They share the same store and ACL, but they are
shaped for different jobs — so agents and humans write and recall them through
different tools.

| Kind | One-line job |
| --- | --- |
| **Entry** | A durable *record* — something that happened, was decided, or is true |
| **Guide** | Reusable *advice* — when situation X occurs, do Y (and avoid Z) |

Same insight can be both: save the fact as an entry **and** the standing rule
as a guide when you need the record *and* the procedure later.

## Spaces

Memories live in a **space**: your **personal** space (private, created with
the account) or an optional shared **group**. Omit `space_id` on writes →
personal. Groups, membership, and write-target resolution:
{doc}`groups`.

## Entries

An **entry** is a typed durable record. Use it when you care about *what is
true*, *what was chosen*, or *what was learned* — not when you mainly need a
repeatable “if this, then that” playbook (that is a guide).

### What makes a good entry

- It stays useful after this chat ends.
- A future agent (or you) would otherwise rediscover it the hard way.
- It is concrete: a named decision, a root cause, a preference, a path quirk.

Skip transient status, todos, raw logs, guesses, secrets, and facts that are
already obvious from the code alone.

### How entries work

| Concern | Behavior |
| --- | --- |
| Create | MCP `remember` (or the matching REST/GUI create) |
| Near-duplicates | **Refused** when title+body similarity is at or above
  `NEAR_DUP_THRESHOLD` (default `0.92`) in the same space — pass `force` to
  create anyway |
| Classification | Closed **type** enum (below). Types are labels for filtering and
  agent judgment — the server does **not** run a different pipeline per type |
| Path scoping | Optional `path_hint` / `repo_hint` so `about_path` and filtered
  `find` can surface code-tied memories |
| Expiry | Optional `expires_at` metadata; search does **not** hide expired rows —
  extend or clear via `revise` |
| Patch / delete / move | `revise`, `forget`, `relocate` |
| Recall | MCP `find` defaults to **entries only**; digests first, then `open`
  for the full body |

Shared tools that also apply to guides: `open`, `revise`, `forget`, `browse`,
`since`, `by_tag`, `relocate`, `trace`, `about_path`, `overlap_report`.

### Entry types

Pick the best label on every `remember`. Prefer a specific type over the API
default `note`.

| Type | Use when | Example |
| --- | --- | --- |
| `fact` | Stable truth about the world or codebase | “Compose project name is `memchentic`; services are `postgres` and `application`.” |
| `preference` | How someone wants work done | “Never put localhost in user-facing URLs; resolve a host address first.” |
| `decision` | A choice and its rationale | “Live memory bus uses Postgres `NOTIFY` on channel `memchentic_memory` — no Redis.” |
| `instruction` | Standing always/never directive for agents | “Do not commit or push unless the user asks.” |
| `feedback` | Correction after a mistake | “User ruled out legacy shims — hard-cut only on API changes.” |
| `insight` | Learned non-obvious finding (bug, quirk, workflow) | “Pytest in the app container always uses DB `memchentic_test`, never the live volume.” |
| `note` | Catch-all when nothing else fits | Short scratch that still deserves persistence |

Guides are **not** an entry type. Do not invent a type named `guide` on
`remember` — use `guide_write`.

## Guides

A **guide** is reusable procedural advice. Use it when the value is *how to
act next time*, especially in a **when X → do Y / avoid Z** shape.

### What makes a good guide

- It generalizes beyond one incident.
- An agent can apply it without reopening the original ticket or chat.
- Re-stating the same advice later should *strengthen* it, not create a twin
  row.

Typical bodies read like:

```text
When X = <situation or trigger>:
do Y = <preferred action>
Avoid Z = <failure mode or anti-pattern>

Context: <optional short evidence or verification note>
```

### How guides work

| Concern | Behavior |
| --- | --- |
| Create / reinforce | MCP `guide_write` |
| Near-duplicates | **Upsert**: matching guide in the same space is updated; its
  **confidence** increases by `1.0` (starts at `1.0` on first write). There is
  no `force` flag |
| Classification | No entry `type` — `entry_type` is null; kind is `guide` |
| Path / expiry knobs | Not on `guide_write` (path hints and expiry are
  remember-oriented; patch via `revise` if needed) |
| Ranking | `guide_find` hybrid-searches guides, then boosts score by confidence
  (roughly hybrid score × `(1 + 0.1 × confidence)`) so reinforced advice rises |
| Patch / delete | Same as entries: `revise` / `forget` |

Use `guide_find` when you want advice. Use `find` when you want typed records.
Agents usually call **both** at the start of meaningful work.

## Entry vs guide — choose deliberately

| Question | Prefer entry | Prefer guide |
| --- | --- | --- |
| What am I storing? | A record of truth, preference, or outcome | A playbook for a recurring situation |
| Will I quote it as “this is so”? | Yes | Sometimes, but mainly “do this when…” |
| Should repeats create a new row? | No — refuse unless `force` | No — merge and raise confidence |
| Do I need a type label for filters? | Yes (`fact`, `decision`, …) | No |
| Code-path recall (`about_path`)? | First-class via `path_hint` / `repo_hint` | Secondary (body may still mention paths) |
| Primary search tool | `find` | `guide_find` |

**Write both** when the same insight is a durable fact *and* a standing rule
(for example: an architecture decision entry plus a guide that says how to
extend that subsystem without breaking the rule).

**Write neither** for one-shot chatter, unverified guesses, or anything that
must not leave the session (secrets, tokens, private personal data).

## Search and embeddings

Vectors are derived from title + body on write (hardcoded MiniLM ONNX, 384-d).
Search is **hybrid**: full-text plus pgvector, fused for ranking. Tools return
**digests** (id, title, score, snippet, kind/type, tags, space). Call `open`
for the full body.

Path-scoped recall uses `about_path` with a path and optional repo hint
(matches `path_hint` or body text). Hygiene without auto-delete:
`overlap_report`, then merge via `revise` / `forget`.

Tool inventory and the suggested agent loop: {doc}`mcp-tools`.
