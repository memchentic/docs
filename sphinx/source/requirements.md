# Requirements

What you need before you bring Memchentic up. Pick one path
({doc}`docker` or {doc}`helm`); both run the same application against Postgres
with vector search.

## Deploy path

| Path | You need |
| --- | --- |
| Docker Compose | Docker Engine with Compose v2, and free host ports for the web UI and agent connection |
| Kubernetes (Helm) | A cluster you can install into, plus Helm 3 |

## Runtime pieces

| Piece | Requirement |
| --- | --- |
| Application image | `registry.memchentic.ai/memchentic/memchentic:latest` (preferred; or a version tag). Mirrored on GHCR. |
| Database | Postgres **16** with **pgvector** (Compose and the Helm chart default to `pgvector/pgvector:0.8.7-pg16`) |
| Storage | Durable disk for Postgres (a Compose volume or a Kubernetes PVC) |

The first start prepares the database. Point the app at the database with
`POSTGRES_HOSTNAME` (or the Helm equivalent) — in-cluster Postgres, the
Compose database service, or a host you already run.

## Secrets

These must be set before a healthy start. Weak placeholders and short values
are rejected.

| Secret | Rule |
| --- | --- |
| `ADMIN_PASSWORD` | At least **16** characters; not a placeholder such as `change-me` |
| `ADMIN_API_KEY` | At least **16** characters; not a placeholder such as `change-me-admin-api-key` |
| `POSTGRES_PASSWORD` | Non-empty password for the database role |

`ADMIN_EMAIL` / `ADMIN_DISPLAY_NAME` create the first admin user (defaults are
fine for a first bring-up). Full list: {doc}`environment-variables`.

## Network and clients

| Piece | Notes |
| --- | --- |
| Public URL | Set `BASE_URL` and `CORS_ORIGINS` to the address people open in the browser |
| Compose ports (defaults) | one port, `18080`, for the web UI, HTTP API (`/api`), and agents / MCP (`/mcp`) |
| Kubernetes | Service on **8080**; Ingress is off by default |
| Coding agents | Cursor, Claude Code, Copilot, Codex, or another MCP client after the stack is up — see {doc}`connect-mcp` |

Login defaults to built-in email and password. An external provider is
optional; when you use one, register the callback URL from
{doc}`environment-variables` on that provider.

## Optional extras

| Extra | When |
| --- | --- |
| External Postgres | You already run Postgres 16 + pgvector elsewhere |
| External login provider | Google, GitHub, Okta, Authentik, Keycloak, Auth0, Entra, GitLab, or another OIDC provider |
| GHCR login | Only if image pulls are private in your environment |
| Marketplace plugin | Optional after MCP works — {doc}`install-marketplace` |

## Next step

- Compose: {doc}`docker`
- Kubernetes: {doc}`helm`
