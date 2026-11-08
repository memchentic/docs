# Environment Variables

Settings come from the process environment (Compose, Kubernetes, or your
host). There is no separate `DATABASE_URL` — database pieces are set with the
`POSTGRES_*` variables below.

Values marked **required at start** refuse weak placeholders and will not boot
until you set them.

## Compose ports and image

These control how Docker Compose publishes the stack. They select the image and
host ports; they are not application feature toggles.

| Variable | Default | Description |
| --- | --- | --- |
| `COMPOSE_FILE` | `docker-compose.yaml` | Which Compose files to load. |
| `APPLICATION_IMAGE` | `registry.memchentic.ai/memchentic/memchentic:latest` | Image for the Memchentic service (preferred registry; GHCR mirror available). |
| `GUI_PORT` | `18080` | Host port for the web UI. |
| `API_PORT` | `18081` | Host port for the HTTP API. |
| `MCP_PORT` | `18082` | Host port for agent / MCP connections. |

## Public URL and logging

| Variable | Default | Description |
| --- | --- | --- |
| `BASE_URL` | `http://localhost:8080` | Public origin browsers and login callbacks use. |
| `CORS_ORIGINS` | `http://localhost:5173,http://127.0.0.1:5173` | Comma-separated allowed browser origins. Set this to your real UI URL in production. |
| `LOG_LEVEL` | `INFO` | Log level (`DEBUG`, `INFO`, `WARNING`, `ERROR`, …). |

## Admin bootstrap

| Variable | Default | Description |
| --- | --- | --- |
| `ADMIN_EMAIL` | `admin@example.com` | Email for the first admin account. |
| `ADMIN_DISPLAY_NAME` | `Admin` | Display name for that admin. |
| `ADMIN_PASSWORD` | *(must set)* | **Required at start.** At least 16 characters; not a known placeholder. |
| `ADMIN_API_KEY` | *(must set)* | **Required at start.** Break-glass API key for that admin. Prefer a personal API key from the UI for everyday agents. |

## Login / identity

| Variable | Default | Description |
| --- | --- | --- |
| `OAUTH_PROVIDER` | `internal` | `internal` (built-in email/password) or an external provider: `google`, `github`, `okta`, `authentik`, `keycloak`, `auth0`, `entra`, `gitlab`, `generic`. |
| `OAUTH_CLIENT_ID` | *(empty)* | Client id from the external provider. |
| `OAUTH_CLIENT_SECRET` | *(empty)* | Client secret. May be empty for `internal`. |
| `OAUTH_ISSUER` | *(empty)* | Issuer URL for an external provider. |
| `OAUTH_AUDIENCE` | *(empty)* | Expected audience claim when verifying provider tokens. |
| `OAUTH_JWKS_URI` | *(empty)* | JWKS URL for verifying provider tokens. |
| `OAUTH_SCOPES` | `openid,profile,email` | Scopes requested from the provider. |

When `OAUTH_PROVIDER` is not `internal`, register this redirect URI on the
provider:

```text
{BASE_URL}/api/auth/oauth/callback
```

## Registration and sessions

| Variable | Default | Description |
| --- | --- | --- |
| `REGISTER_ENABLED` | `true` | Allow public self-registration for built-in login. Also toggleable later in System Settings. |
| `SESSION_REMEMBER_DAYS` | `30` | Lifetime (days) for “remember me” sessions. Minimum `1`. |
| `SESSION_BROWSER_HOURS` | `12` | Lifetime (hours) for normal browser sessions. Minimum `1`. |
| `SESSION_COOKIE_SECURE` | *(derived)* | Force secure cookies: `true` / `false`. Leave unset to follow the `BASE_URL` scheme. |

## Database

| Variable | Default | Description |
| --- | --- | --- |
| `POSTGRES_USER` | `memchentic` | Database user. |
| `POSTGRES_PASSWORD` | `memchentic` | Database password. |
| `POSTGRES_HOSTNAME` | `postgres` | Database hostname. |
| `POSTGRES_PORT` | `5432` | Database port. |
| `POSTGRES_DB` | `memchentic` | Database name. |

Postgres must support the `vector` extension (for example
`pgvector/pgvector:pg16`).

## Search defaults

| Variable | Default | Description |
| --- | --- | --- |
| `NEAR_DUP_THRESHOLD` | `0.92` | How similar a new write must be before it is treated as a near-duplicate. |
| `RRF_K` | `60` | Ranking blend factor for hybrid search. |
| `SEARCH_DEFAULT_LIMIT` | `20` | Default number of search results when the client does not ask for a limit. |
| `BROWSE_DEFAULT_LIMIT` | `50` | Default number of browse results when the client does not ask for a limit. |
| `RESULT_MAX_LIMIT` | `100` | Maximum results allowed for search or browse. |

See also {doc}`docker` and {doc}`helm`.
