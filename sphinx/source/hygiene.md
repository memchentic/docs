# Hygiene

Over time a space collects near-copies, replaced advice that was never
cleaned up, and memories that are about to expire. **Hygiene** is how
Memchentic surfaces those cases so you (or an agent) can act on them —
nothing is deleted unless you, or an admin automation you enabled, say so.

The **Hygiene** page in the web UI shows the same report as the MCP tool
`overlap_report`.

## Supersedes

When a newer entry or guide replaces an older one in the **same space**, the
writer can declare that link:

- MCP: `remember`, `guide_write`, or `revise` with `supersedes` set to the
  older memory’s id (`supersedes=""` on `revise` clears the link)
- REST: the same field on create/update bodies

Effects:

- Search and browse **hide** the superseded target while the link is live
  (both ends still exist). The replacement stays visible.
- `overlap_report` lists the pair under **conflicts** (`reason`:
  `supersedes`) until someone resolves it.
- Full opens still return both ids; the replacement carries
  `supersedes_id`.

## Resolving by hand

| Action | MCP | What it does |
| --- | --- | --- |
| Accept a supersedes conflict | `resolve_conflict` (`a` = replacement, `b` = superseded) | Forgets `b` when `a` still points at it |
| Keep one near-duplicate | `resolve_near_duplicate` (`keep` / `drop`) | Forgets `drop` when the pair still qualifies as near-duplicates |

Both use the same ACL as `forget`: the memory’s creator, the **group
creator**, or an instance admin. A group admin role alone is not enough.

The GUI Hygiene page offers the same actions under **Conflicts** and
**Near-duplicates**. A third section lists soon-to-expire memories (bounded
by `expiring_within_days` on `overlap_report`).

## Consolidation proposals

When **Propose consolidations** is on (see below), `overlap_report` also
returns a `proposals` list derived from near-duplicate pairs. Each proposal
names a `keep` id (newer row) and a `drop` id (older row) with
`action: merge`. The GUI Hygiene page shows the same list.

Resolve consolidations the same way as near-duplicates: `resolve_near_duplicate`
with those ids, or the matching control on the Hygiene page. Auto-apply (below)
runs the same merge during a scheduled pass.

## Automation (admin)

Instance administrators can turn on scheduled cleanup under **System
Settings → Automation**:

| Setting | Effect |
| --- | --- |
| Auto-resolve conflicts | Forgets the superseded side of live `supersedes` pairs |
| Auto-resolve near-duplicates | Forgets the older memory in each near-duplicate pair |
| Propose consolidations | Adds merge proposals to `overlap_report` and the Hygiene UI |
| Auto-apply consolidations | During a manual or scheduled pass, merges pending proposals (keep newer, forget older) |

Passes run in the background (about every 15 minutes after a short boot
delay) and cap how many rows they touch per pass. Admins can also run one
pass immediately from that panel. Both toggles default to **off**.

## Related

- Ranking, near-dup refuse on write, and lifetime: {doc}`memory`
- Tool inventory: {doc}`mcp-tools`
- Mail and other system settings: {doc}`environment-variables`
