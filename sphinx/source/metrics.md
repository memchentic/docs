# Metrics

The **Metrics** page summarises storage and retrieval for the spaces you can
read. It is a human dashboard — agents do not need it day to day.

## What it shows

Within a time window (`7d` / `30d` / `90d` / `all`, or an absolute from/to):

- Corpus size and mix (entries vs guides, never-accessed, and related KPIs)
- Creations and accesses over time
- Breakdowns by type, tags, space, and project
- Top recalled memories and accesses by source (search, open, reinforce, …)

Recall events are stored when a live memory is touched by search/`open` (with
debounce) or when a guide is reinforced as a near-duplicate. Opening a memory
in full (GUI, REST, or MCP `open`) also returns a short `recent_recalls` list
alongside `recall_count` and `last_recalled_at`.

## Shared filters

**Memories** and **Metrics** share the same filter language: space, scope,
kind, types, tags, project, and a title contains query. Narrow the corpus
once and the charts follow the same slice.

API surface: `GET /api/metrics/memories` (Bearer auth; limited to spaces the
caller can read).

## Related

- How recall renews lifetime and ranks results: {doc}`memory`
- Projects as a filter facet: {doc}`projects`
