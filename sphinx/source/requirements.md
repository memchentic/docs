# Requirements

What you need before you bring Memchentic up. Pick one deploy path
({doc}`docker` or {doc}`helm`); both run the same application image against
Postgres with pgvector.

## Deploy path

| Path | You need |
| --- | --- |
| Docker Compose | Docker Engine with Compose v2, and enough host ports free for the GUI / API / MCP publishes |
| Kubernetes (Helm) | A cluster you can install into, plus Helm 3 |

## Runtime components

| Component | Requirement |
| --- | --- |
| Application image | `ghcr.io/memchentic/memchentic:latest` (one process on `:8080` — GUI `/`, API `/api`, MCP `/mcp`; pin a semver if you need a fixed digest) |
| Database | Postgres **16** with **pgvector** (Compose and the Helm chart default to `pgvector/pgvector:pg16`) |
| Persistent volume | Durable storage for Postgres data (Compose named volume or a Kubernetes PVC) |

Migrations run automatically when the application starts. Point
`POSTGRES_HOSTNAME` (or Helm `config.postgresHostname`) at the database the
app should use — in-cluster chart Postgres, the Compose `postgres` service, or
your own external host.

## Secrets and bootstrap

These must be set before a healthy boot. Weak placeholders and short values
are rejected at start.

| Secret | Rule |
| --- | --- |
| `ADMIN_PASSWORD` | At least **16** characters; not a known placeholder such as `change-me` |
| `ADMIN_API_KEY` | At least **16** characters; not a known placeholder such as `change-me-admin-api-key` |
| `POSTGRES_PASSWORD` | Non-empty password for the database role |

`ADMIN_EMAIL` / `ADMIN_DISPLAY_NAME` bootstrap the break-glass admin user
(defaults are fine for a first bring-up). Full variable reference:
{doc}`environment-variables`.

## Network and clients

| Piece | Notes |
| --- | --- |
| Public origin | Set `BASE_URL` and `CORS_ORIGINS` to the URL browsers and OAuth callbacks use |
| Host ports (Compose defaults) | `GUI_PORT` 18080 → `/`, `API_PORT` 18081 → `/api`, `MCP_PORT` 18082 → `/mcp` |
| Kubernetes Service | ClusterIP on **8080**; Ingress is off by default |
| MCP client | Cursor, Claude Code, Copilot, Codex, or another MCP client pointed at `/mcp` after the stack is up |

Identity defaults to `OAUTH_PROVIDER=internal` (email/password in Memchentic).
An external IdP is optional; when you use one, register
`{BASE_URL}/api/auth/oauth/callback` on that provider.

## Optional extras

| Extra | When |
| --- | --- |
| External Postgres | You already run Postgres 16 + pgvector elsewhere (disable chart Postgres or point Compose `POSTGRES_HOSTNAME` at it) |
| External OAuth IdP | Google, GitHub, Okta, Authentik, Keycloak, Auth0, Entra, GitLab, or a generic OIDC provider |
| GHCR login | Only if the image pull is private for your environment |
| Marketplace plugin | Optional Cursor (or similar) plugin install after MCP works — see {doc}`install-marketplace` |

## Next step

- Compose: {doc}`docker`
- Kubernetes: {doc}`helm`
