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
| `GUI_PORT` | `18080` | Host port for the web UI, HTTP API (`/api`), and MCP (`/mcp`). |
| `BIND_ADDRESS` | `127.0.0.1` | Host address the port binds to. Use `0.0.0.0` to accept connections from other machines. |
| `APPLICATION_MEMORY_LIMIT` | `2g` | Memory limit for the Memchentic container. |
| `RUN_MIGRATIONS_ON_BOOT` | `true` | Run database migrations when the app starts. Set `false` when migrations run separately (`python -m memchentic.migrate`, the `migrate.yaml` Compose overlay, or the Helm `migrations.initContainer` option). |

## Public URL and logging

| Variable | Default | Description |
| --- | --- | --- |
| `BASE_URL` | `http://localhost:8080` | Public origin browsers and login callbacks use. |
| `CORS_ORIGINS` | `http://localhost:5173,http://127.0.0.1:5173` | Comma-separated allowed browser origins. Set this to your real UI URL in production. |
| `LOG_LEVEL` | `INFO` | Log level (`DEBUG`, `INFO`, `WARNING`, `ERROR`, …). |

## Admin bootstrap

| Variable | Default | Description |
| --- | --- | --- |
| `ADMIN_EMAIL` | `admin@example.com` | Email for the break-glass admin account. Changing it demotes the previous break-glass account to an ordinary user at the next start, and revokes the sessions and API keys the newly named account held before. |
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
| `OAUTH_NATIVE_REDIRECT_SCHEMES` | `cursor` | Comma-separated private-use URI schemes (RFC 8252) that native MCP clients may register as OAuth redirect URIs. |
| `OAUTH_SCOPES` | *(blank)* | Comma-separated scopes requested from the provider. Blank uses the provider preset default (`read:user,user:email` for GitHub, `openid,profile,email,offline_access` for Entra, `openid,profile,email` otherwise). |

When `OAUTH_PROVIDER` is not `internal`, register this redirect URI on the
provider:

```text
{BASE_URL}/api/auth/oauth/callback
```

## Registration and sessions

| Variable | Default | Description |
| --- | --- | --- |
| `REGISTER_ENABLED` | `true` | Allow public self-registration for built-in login. With an external `OAUTH_PROVIDER` it gates first-time single sign-on instead; an auth-group IdP bind always admits its members, and a bound Users auth group admits only its members. First-boot value only; afterwards System Settings is the source of truth. |
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
| `SEARCH_RECENCY_HALF_LIFE_DAYS` | `30` | Age in days at which a memory's recency boost drops to half. |
| `SEARCH_RECENCY_WEIGHT` | `0.25` | Largest recency boost (0 to 1). A memory edited now scores at most `1 +` this times an equally relevant old one. `0` turns recency off. |
| `SEARCH_PATH_BOOST` | `0.15` | Extra weight for memories whose path hint matches the search path. |
| `SEARCH_MIN_SIMILARITY` | `0.2` | Lowest meaning similarity (-1 to 1) at which a memory that matches no query keyword is still returned. `-1` turns the floor off. |
| `NEAR_DUP_CANDIDATES` | `5` | How many nearest existing memories are checked for a near-duplicate on write. |
| `OVERLAP_SCAN_LIMIT` | `2000` | How many recent memories the overlap report scans. |

| `MEMORY_USAGE_SCORE_WEIGHT` | `0.05` | Extra weight for memories that are recalled often. |

## Memory lifetime

| Variable | Default | Description |
| --- | --- | --- |
| `MEMORY_DEFAULT_TTL_DAYS` | `90` | Days a new memory lives when the caller does not choose a lifetime. `0` makes new memories permanent by default. |
| `MEMORY_TTL_EXTEND_DAYS` | `90` | Each recall pushes a live memory's expiry to at least this many days from now. |
| `MEMORY_TTL_TOUCH_DEBOUNCE_HOURS` | `6` | A memory is renewed and counted at most once per this window. |
| `MEMORY_MAX_TTL_DAYS` | `730` | Renewal never pushes expiry past this many days after creation. `0` removes the ceiling. |

How these combine into a ranking and a lifetime is explained in {doc}`memory`.

See also {doc}`docker` and {doc}`helm`.
