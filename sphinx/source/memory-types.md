# Memory Types

Memchentic stores memories in **spaces**. Every memory is one of two **kinds**:
an **entry** or a **guide**. They share the same store and ACL, but they are
shaped for different jobs.

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

### Entry types

Every entry has one closed **type**. Types are labels for filtering and
judgment — the server does **not** run a different pipeline per type. Prefer a
specific type over the default `note`.

| Type | Use when | Example |
| --- | --- | --- |
| `fact` | Stable truth about the world or codebase | “Compose project name is `memchentic`; services are `postgres` and `application`.” |
| `preference` | How someone wants work done | “Never put localhost in user-facing URLs; resolve a host address first.” |
| `decision` | A choice and its rationale | “Live memory bus uses Postgres `NOTIFY` on channel `memchentic_memory` — no Redis.” |
| `instruction` | Standing always/never directive for agents | “Do not commit or push unless the user asks.” |
| `feedback` | Correction after a mistake | “User ruled out legacy shims — hard-cut only on API changes.” |
| `insight` | Learned non-obvious finding (bug, quirk, workflow) | “Pytest in the app container always uses DB `memchentic_test`, never the live volume.” |
| `note` | Catch-all when nothing else fits | Short scratch that still deserves persistence |

Guides are **not** an entry type — use a guide when the value is reusable
advice, not a typed record.

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

Repeating a near-identical guide updates the existing one and raises its
**confidence**, so reinforced advice ranks higher on later recall.

## Entry vs guide — choose deliberately

| Question | Prefer entry | Prefer guide |
| --- | --- | --- |
| What am I storing? | A record of truth, preference, or outcome | A playbook for a recurring situation |
| Will I quote it as “this is so”? | Yes | Sometimes, but mainly “do this when…” |
| Do I need a type label (`fact`, `decision`, …)? | Yes | No |
| Should repeats strengthen the same memory? | No — keep distinct records | Yes — merge and raise confidence |

**Write both** when the same insight is a durable fact *and* a standing rule
(for example: an architecture decision entry plus a guide that says how to
extend that subsystem without breaking the rule).

**Write neither** for one-shot chatter, unverified guesses, or anything that
must not leave the session (secrets, tokens, private personal data).

## Search and embeddings

Vectors are derived from title + body on write (hardcoded MiniLM ONNX, 384-d).
Search is **hybrid**: full-text plus pgvector, fused for ranking. Results start
as short **digests**; open the full body when you need the text.

Path-scoped recall can match path/repo hints or body text. Hygiene reports
near-duplicates without auto-deleting — merge or forget deliberately.

Create, search, and patch tool names: {doc}`mcp-tools`.
