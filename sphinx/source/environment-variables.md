# Environment Variables

Memchentic reads configuration from the process environment (Compose
`environment:` / `env_file`, Kubernetes, or your host). There is no separate
`DATABASE_URL` — the app builds the DSN from the `POSTGRES_*` pieces.

Values marked **required at start** refuse weak placeholders and will not boot
until you set them.

## Compose client (repo `docker/.env`)

These affect how Docker Compose runs the stack. They are not read by the
application process itself (except that `APPLICATION_IMAGE` selects which
image Compose starts).

| Variable | Default | Description |
| --- | --- | --- |
| `COMPOSE_FILE` | `docker-compose.yaml` | Compose file list for the sticky `.env` in `docker/`. |
| `APPLICATION_IMAGE` | `ghcr.io/memchentic/memchentic:latest` | Image Compose pulls for the `application` service. |
| `GUI_PORT` | `18080` | Host port published for GUI `/`. |
| `API_PORT` | `18081` | Host port published for API `/api`. |
| `MCP_PORT` | `18082` | Host port published for MCP `/mcp`. |

All three host ports map to the same container listener on `:8080`.

## Application URL and HTTP

| Variable | Default | Description |
| --- | --- | --- |
| `BASE_URL` | `http://localhost:8080` | Public origin browsers and OAuth callbacks use. Also used as the issuer for memchentic-signed session tokens. |
| `CORS_ORIGINS` | `http://localhost:5173,http://127.0.0.1:5173` | Comma-separated allowed browser origins. |
| `LOG_LEVEL` | `INFO` | Process log level (standard levels such as `DEBUG`, `INFO`, `WARNING`, `ERROR`). |

## Admin bootstrap

| Variable | Default | Description |
| --- | --- | --- |
| `ADMIN_EMAIL` | `admin@example.com` | Login email for the break-glass admin principal. Empty falls back to `admin@example.com`. |
| `ADMIN_DISPLAY_NAME` | `Admin` | Display name for that admin user. |
| `ADMIN_PASSWORD` | *(must set)* | **Required at start.** Password for the admin user. Rejected if empty, a known placeholder (`change-me`), or shorter than 16 characters. |
| `ADMIN_API_KEY` | *(must set)* | **Required at start.** Break-glass API key (same admin principal). Rejected if empty, a known placeholder (`change-me-admin-api-key`), or shorter than 16 characters. Prefer a per-user API key from the GUI for day-to-day agents. |

## OAuth / identity

| Variable | Default | Description |
| --- | --- | --- |
| `OAUTH_PROVIDER` | `internal` | Identity mode. One of: `internal`, `google`, `github`, `okta`, `authentik`, `keycloak`, `auth0`, `entra`, `gitlab`, `generic`. |
| `OAUTH_CLIENT_ID` | *(empty)* | IdP client id. Required for browser “Continue with …” when the provider is not `internal`. |
| `OAUTH_CLIENT_SECRET` | *(empty)* | IdP client secret. May be empty when `OAUTH_PROVIDER=internal`. |
| `OAUTH_ISSUER` | *(empty)* | External IdP issuer URL. Used when the provider is not `internal` (falls back to `BASE_URL` if unset). |
| `OAUTH_AUDIENCE` | *(empty)* | Expected audience claim. Defaults to `{BASE_URL}/mcp` when unset. |
| `OAUTH_JWKS_URI` | *(empty)* | JWKS URL for verifying IdP tokens when using an external provider. |
| `OAUTH_SCOPES` | `openid,profile,email` | Comma-separated scopes requested from the IdP. |

When `OAUTH_PROVIDER` is not `internal`, register this redirect URI on the IdP:

```text
{BASE_URL}/api/auth/oauth/callback
```

The same client id drives MCP harness Authorize and GUI Continue-with.

## Registration and sessions

| Variable | Default | Description |
| --- | --- | --- |
| `REGISTER_ENABLED` | `true` | Allow public self-registration when using `OAUTH_PROVIDER=internal`. Also toggleable later in System Settings. |
| `SESSION_REMEMBER_DAYS` | `30` | Lifetime (days) for “remember me” GUI sessions. Minimum `1`. |
| `SESSION_BROWSER_HOURS` | `12` | Lifetime (hours) for normal browser sessions. Minimum `1`. |
| `SESSION_COOKIE_SECURE` | *(derived)* | Force the session cookie `Secure` flag: `true` / `false`. Leave unset or blank to derive from the `BASE_URL` scheme (`https` → secure). |

## Database

| Variable | Default | Description |
| --- | --- | --- |
| `POSTGRES_USER` | `memchentic` | Database user. |
| `POSTGRES_PASSWORD` | `memchentic` | Database password. |
| `POSTGRES_HOSTNAME` | `postgres` | Hostname of Postgres (Compose service name, or an external host). |
| `POSTGRES_PORT` | `5432` | Database port (`1`–`65535`). |
| `POSTGRES_DB` | `memchentic` | Database name. |

Postgres must support the `vector` extension (for example `pgvector/pgvector:pg16`).

## Search and recall defaults

MiniLM ONNX embeddings are baked into the image — there are no model path or
dimension environment variables.

| Variable | Default | Description |
| --- | --- | --- |
| `NEAR_DUP_THRESHOLD` | `0.92` | Cosine similarity threshold used when refusing near-duplicate writes. |
| `RRF_K` | `60` | Reciprocal rank fusion `k` for hybrid search. |
| `SEARCH_DEFAULT_LIMIT` | `20` | Default `limit` for search/find-style calls when the client omits it. Minimum `1`. |
| `BROWSE_DEFAULT_LIMIT` | `50` | Default `limit` for browse/list-style calls when the client omits it. Minimum `1`. |
| `RESULT_MAX_LIMIT` | `100` | Hard upper clamp applied to any search or browse `limit`. Minimum `1`. |

See also {doc}`docker` and {doc}`helm`.
