# Memory

Memchentic keeps long-lived knowledge in **spaces**. Every memory is one of
two kinds: an **entry** or a **guide**.

| Kind | In plain terms |
| --- | --- |
| **Entry** | Something worth *remembering* — a fact, preference, decision, or lesson |
| **Guide** | Something worth *following again* — standing advice for a recurring situation |

They live in the same places and share the same privacy rules. The difference
is the job each one is for.

The same idea can show up as both when you want the record *and* the playbook
(for example a decision you made, plus the habit you want next time).

## Spaces

Memories belong to a **space**: your private **personal** space, or an
optional shared **group**. Personal is the default. Shared groups are covered
in {doc}`groups`.

## Entries

An **entry** is a durable record. Save one when the important part is *what
is true*, *what you chose*, or *what you learned*.

Good entries tend to:

- Still matter after the current conversation ends
- Save someone from rediscovering the same hard-won detail later
- Name something concrete — a preference, a decision, a root cause, a quirk

Skip chatter, unfinished todos, raw dumps, guesses, and secrets. Skip things
that are already obvious without storing them.

### Entry types

Every entry has a **type**. The type is a label so you can filter and scan
memories more easily — it does not change how search works under the hood.

| Type | Meaning | Example |
| --- | --- | --- |
| `fact` | A stable truth | “Production publishes the GUI on port 18080.” |
| `preference` | How someone wants work done | “Prefer short answers unless detail is asked for.” |
| `decision` | A choice and why | “Docs ship as their own image, separate from the app.” |
| `instruction` | A standing always / never rule | “Never commit unless asked.” |
| `feedback` | A correction after a mistake | “Hard-cut API changes — no legacy shims.” |
| `insight` | A non-obvious lesson | “The test database is separate from the live one.” |
| `note` | Anything that does not fit cleanly above | A short durable scratch note |

A guide is its own kind of memory, not an entry type. If what you need is
reusable advice rather than a labeled record, use a guide.

## Guides

A **guide** is reusable advice. Save one when the important part is *what to
do next time* — especially “when this happens, do that, and avoid the other
thing.”

Good guides tend to:

- Apply to more than one incident
- Stay useful without rereading the original conversation
- Get stronger when the same advice is confirmed again, instead of spawning
  near-duplicate copies

A natural shape looks like:

```text
When this situation comes up:
do this
Avoid that

Optional context: why this advice stuck
```

When similar guide content is saved again, Memchentic updates the existing
guide and raises its **confidence**. Higher confidence means that advice ranks
more strongly when you search later.

## Choosing entry or guide

| If you mainly need… | Choose |
| --- | --- |
| A record of what is true or what was decided | **Entry** |
| A playbook for a situation that will come up again | **Guide** |
| Both the record and the playbook | **Both** |
| Something temporary, unverified, or secret | **Neither** |

## How search ranks memories

When an agent asks memchentic a question, it does not get "the newest
memories" or "the closest vector match." It gets a short list ranked by
**how well each memory matches the question, adjusted for how fresh it is and
how often it has been reinforced and used**. Memories that nobody uses
expire on their own, so the store does not fill up with stale notes.

This page shows the exact formula, so you can see why old knowledge still
surfaces when it matters, why new knowledge is preferred when matches are
close, and why the result list stays short instead of filling up with noise.

### The pipeline at a glance

```text
Question ─┬─> Meaning match ──┐
          │                   ├─> Fuse rankings ─> Adjust for freshness
          └─> Keyword match ──┘                    and reinforcement
                                                         │
                                                         v
                                                  Short ranked list
```

1. **Filter.** Only memories in spaces you can see are considered. Memories
   past their expiry date are skipped. Optional filters (type, tags, path,
   repo) narrow it further.
2. **Find candidates two ways.** One list ranks memories by *meaning*
   (semantic vectors). A second list ranks them by *words* (Postgres full-text
   search, with titles weighted above bodies).
3. **Fuse.** The two lists are combined into one score per memory.
4. **Adjust.** That score is multiplied by freshness, path alignment,
   confidence (for guides), and usage.
5. **Return the top few.** Default 20 results; never more than 100.

### The formula

For a memory $m$ and a query $q$:

$$
\text{score}(m) =
\underbrace{\text{match}(m, q)}_{\text{how well it answers}}
\times
\underbrace{\text{freshness}(m)}_{\text{how current}}
\times
\underbrace{\text{path}(m)}_{\text{where it applies}}
\times
\underbrace{\text{confidence}(m)}_{\text{how reinforced}}
\times
\underbrace{\text{usage}(m)}_{\text{how often recalled}}
$$

Each piece is explained below.

#### Match: reciprocal rank fusion

memchentic ranks candidates by meaning and by keywords separately, then fuses
the two **rankings** (not the raw similarity numbers):

$$
\text{match}(m, q) = \sum_{\text{list} \,\in\, \{\text{meaning},\ \text{keywords}\}}
\frac{1}{k + \text{rank}_{\text{list}}(m)}
$$

- $\text{rank}$ starts at 1 for the best hit in each list. A memory missing
  from a list adds nothing for that list.
- $k$ is `RRF_K` (default **60**). A larger $k$ flattens the gap between
  rank 1 and rank 10.

**Why this matters:** a memory that is both *about the right thing* and *uses
the right words* appears in both lists and scores roughly twice as high as a
memory that only matches one way. Paraphrased questions still find the right
memory through meaning; exact names, error strings, and file paths still win
through keywords.

#### Freshness: gentle recency decay

$$
\text{freshness}(m) = \frac{1}{1 + \dfrac{\text{age in days}}{H}}
$$

- Age is measured from the memory's last update (or creation, if never
  edited).
- $H$ is `SEARCH_RECENCY_HALF_LIFE_DAYS` (default **30**).

| Age | Freshness (H = 30) |
| --- | --- |
| Today | 1.00 |
| 30 days | 0.50 |
| 90 days | 0.25 |
| 180 days | 0.14 |
| 1 year | 0.08 |

The curve drops quickly at first and then flattens. Old memories are
discounted, **never zeroed out**. An old memory that is a much stronger match
can still outrank a new, weak one.

Editing a memory (for example with `revise`) resets its age. Keeping a
memory accurate keeps it current.

#### Path: work-location alignment

$$
\text{path}(m) =
\begin{cases}
1 + B & \text{if the query's path hint matches the memory's path hint} \\
1 & \text{otherwise}
\end{cases}
$$

$B$ is `SEARCH_PATH_BOOST` (default **0.15**). Memories saved with a
`path_hint` are favored when an agent searches from that part of a codebase.

#### Confidence: reinforced guides

$$
\text{confidence}(m) =
\begin{cases}
1 + 0.1 \times c & \text{if } m \text{ is a guide with confidence } c \\
1 & \text{if } m \text{ is an entry}
\end{cases}
$$

A new guide starts at $c = 1$. Every time an agent writes nearly the same
guide again, memchentic merges it into the existing guide and raises $c$ by
1 instead of creating a copy. A guide confirmed five times gets a 1.5x boost.
Advice that keeps proving useful rises; one-off advice does not.

#### Usage: memories people actually recall

$$
\text{usage}(m) = 1 + W \times \ln(1 + n)
$$

- $n$ is how many times the memory has been recalled: returned by a search or
  opened. Repeated hits within a few hours count once.
- $W$ is `MEMORY_USAGE_SCORE_WEIGHT` (default **0.05**).

| Recalls | Usage boost |
| --- | --- |
| 0 | 1.00 |
| 5 | 1.09 |
| 50 | 1.20 |
| 500 | 1.31 |

The logarithm keeps this gentle. A popular memory gets a nudge, but it cannot
outrank a far better match just because it is popular. Being recalled does
**not** reset freshness. Only editing does.

### A worked example

Four memories match the same question:

| Memory | Matches | Age | Kind |
| --- | --- | --- | --- |
| A | Rank 1 by meaning and by keywords | 10 days | Entry |
| B | Rank 3 by meaning only | 1 day | Entry |
| C | Rank 2 by meaning and by keywords | 1 year | Entry |
| D | Rank 3 by meaning and by keywords | 45 days | Guide, confidence 4 |

With the defaults ($k = 60$, $H = 30$):

| Memory | Match | x Freshness | x Confidence | Score | Place |
| --- | --- | --- | --- | --- | --- |
| A | 2/61 = 0.0328 | 0.75 | 1.0 | **0.0246** | 1st |
| D | 2/63 = 0.0317 | 0.40 | 1.4 | **0.0178** | 2nd |
| B | 1/63 = 0.0159 | 0.97 | 1.0 | **0.0154** | 3rd |
| C | 2/62 = 0.0323 | 0.08 | 1.0 | **0.0025** | 4th |

What this shows:

- **A** wins because it is both a strong match and recent.
- **D** is older than B and slightly weaker, but it is a guide that has been
  confirmed four times. Reinforcement keeps it above a brand-new, one-sided
  match.
- **B** is the newest memory, but it only matches one way. Freshness alone
  does not put it on top.
- **C** is a strong match but has not been touched in a year. It sinks
  instead of crowding the top results. It is still there: a more specific
  question, or a quick `revise` to confirm it is still true, brings it back.

### Why this works long-term

Large, long-lived memory stores usually fail in one of two ways. They either
forget useful old knowledge, or they bury the answer under years of stale
notes. This formula avoids both.

| Problem | How the formula handles it |
| --- | --- |
| Old but still-correct knowledge disappears | Freshness never reaches zero. A strong match survives age. |
| Stale notes crowd out current ones | Freshness halves at 30 days and keeps falling. Close calls go to the newer memory. |
| Paraphrased questions miss | The meaning list finds memories without matching words. |
| Exact names and error strings get fuzzy results | The keyword list rewards exact terms. Titles count more than bodies. |
| Repeated advice piles up as duplicates | Guides merge and gain confidence instead of copying. |
| Repeated facts pile up as duplicates | Near-identical entries are refused at write time (`NEAR_DUP_THRESHOLD`). |
| Temporary notes linger forever | Every memory expires unless it keeps being used. Agents give scratch notes short lifetimes. |
| Popular knowledge gets cleaned up by mistake | Each recall renews the expiry, and usage nudges the ranking up. |
| Agents flood their context | Results are capped (default 20, max 100), and each one is a short digest until opened. |
| Other teams' notes leak in | Only spaces you belong to are searched. See {doc}`groups`. |

### Tuning

All knobs are environment variables on the application. See
{doc}`environment-variables` for the full list.

| Variable | Default | Turn it up when | Turn it down when |
| --- | --- | --- | --- |
| `SEARCH_RECENCY_HALF_LIFE_DAYS` | `30` | Knowledge stays true for months (policies, architecture) | Facts change weekly |
| `RRF_K` | `60` | You want matches across both lists to matter more evenly | You want top-ranked hits to dominate |
| `SEARCH_PATH_BOOST` | `0.15` | Path-scoped memories should win harder | Paths are noisy or unused |
| `NEAR_DUP_THRESHOLD` | `0.92` | Too many legitimate variants are refused | Near-copies are slipping through |
| `SEARCH_DEFAULT_LIMIT` | `20` | Humans browse results in the UI | Agents waste context on extra results |
| `MEMORY_USAGE_SCORE_WEIGHT` | `0.05` | Frequently used memories should rank higher | Popularity is skewing results |
| `MEMORY_DEFAULT_TTL_DAYS` | `90` | Knowledge is slow to be revisited | The store grows faster than it is used |
| `MEMORY_TTL_EXTEND_DAYS` | `90` | Memories used occasionally should stick | Only frequently used memories should stick |
| `MEMORY_MAX_TTL_DAYS` | `730` | Hot memories should live longer (or `0` for no limit) | Retention policy requires periodic review |

A good starting point for a large organization is a half-life of 60 to 90
days, with a lower default limit (around 10) for agent clients.

## Lifetime: sliding expiry

Ranking (above) decides what comes *first*. Lifetime decides what is *still there*.

Every new memory gets an expiry date:

| The saving agent passes | Expiry |
| --- | --- |
| Nothing | 90 days from now (`MEMORY_DEFAULT_TTL_DAYS`) |
| `ttl_days: 14` | 14 days from now |
| `ttl_days: 0` or `permanent: true` | Never |
| `expires_at` | That exact time |

Agents pick the lifetime when they save: a short one for a temporary
workaround, the default for an ordinary fact, permanent for a standing rule.

**Each recall renews a live memory.** When a memory is returned by a search or
opened, its expiry moves out to at least 90 days from now
(`MEMORY_TTL_EXTEND_DAYS`):

$$
\text{expires}_{\text{new}} = \max\Big(\text{expires},\ \min\big(\text{now} + E,\ \text{created} + M\big)\Big)
$$

- $E$ is `MEMORY_TTL_EXTEND_DAYS` (default **90**).
- $M$ is `MEMORY_MAX_TTL_DAYS` (default **730**, about two years; `0` removes
  the ceiling).
- Renewal never shortens an expiry the agent chose.
- Repeated recalls within `MEMORY_TTL_TOUCH_DEBOUNCE_HOURS` (default **6**)
  count once, so one busy session cannot keep junk alive.
- Writing the same guide again also renews it.

**What happens at expiry.** An expired memory drops out of search, browse, and
the other recall tools. It is not deleted: you can still list it with
`include_expired`, open it by id, and bring it back with `revise` (set a new
`expires_at` or clear the expiry). Recalling an expired memory does not revive
it on its own.

The result: a memory that people keep using stays indefinitely (up to the
ceiling), and a memory nobody has needed for three months leaves search
without anyone having to clean it up.
