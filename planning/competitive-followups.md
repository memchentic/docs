# Competitive follow-ups (deferred)

Planning notes for memchentic as a **general agentic memory** product — not
only coding agents. These items are intentionally **out of scope** for the
current self-hosted MCP release; they inform roadmap conversations without
committing to timelines.

## Graph memory

**Idea:** Treat memories as nodes and explicit links (supersedes, relates-to,
same-project, same-customer) as edges; traverse or summarize neighborhoods for
recall instead of flat hybrid search alone.

**Why defer:** Memchentic already models replacement (`supersedes`), hygiene
proposals, and typed entries/guides in Postgres. A graph layer needs a clear
operator story (migration, query API, MCP contract) and must not weaken ACL or
row-level security.

**When to revisit:** Teams ask for multi-hop “what led to this decision?” across
dozens of spaces, or exports into external knowledge graphs.

## Hosted MCP / SaaS

**Idea:** A managed endpoint (`https://…/mcp`) with tenant isolation, billing,
and zero-compose onboarding.

**Why defer:** The product pitch is **operator-owned** data and identity. A hosted
tier implies SLOs, data residency, key custody, and a different support model
than the open Docker/Helm chart.

**When to revisit:** Demand for “try before self-host” without giving up
export/portability, or enterprise buyers who forbid running the stack themselves.

## Embed SDKs (in-app memory)

**Idea:** Language SDKs that call the REST API from application code — not only
from agent harnesses — e.g. `remember` from a support ticket bot or research
pipeline.

**Why defer:** MCP + REST already share `memchentic.services`; SDKs are mostly
ergonomics and versioning policy. Without them, integrators use OpenAPI and
personal API keys.

**When to revisit:** Repeated requests for typed clients (Python/TS/Go), batch
ingest from non-MCP workers, or official patterns for webhook consumers.

## Principles for any follow-up

- **Same ACL everywhere** — no client-supplied user ids; server principals only.
- **No second memory silo** — graph, hosted, or SDK features must read/write the
  same Postgres corpus, not a forked product vocabulary.
- **Export stays honest** — bounded `export` and standard rows remain the
  portability floor before adding proprietary graph or hosted-only APIs.