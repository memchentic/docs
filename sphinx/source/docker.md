# Docker

Pull the published image from GHCR and run it with Postgres + pgvector.

Image: `ghcr.io/memchentic/memchentic:latest` (semver tags also published; `latest` moves on each release)

## What you need

| Piece | Notes |
| --- | --- |
| Application container | One process on `:8080` — GUI `/`, API `/api`, MCP `/mcp` |
| Postgres 16 + pgvector | Image `pgvector/pgvector:pg16` works; hostname via `POSTGRES_HOSTNAME` |
| Env / secrets | At least strong `ADMIN_PASSWORD` and `ADMIN_API_KEY` (16+ characters each) |

Migrations run on application startup.

## Ready-made Compose in the repo

This repository already ships a Compose stack under `docker/`
(`docker-compose.yaml` plus `.env.example`). You can use that as-is, copy
pieces into your own Compose project, or treat the sketch below as the same
shape without checking out the rest of the tree.

From a clone:

```bash
cd docker
cp .env.example .env
# set strong ADMIN_PASSWORD and ADMIN_API_KEY (16+ characters each)
docker compose up -d
```

Default host publishes (all map to the same container `:8080`):

| Port env | Default | Path |
| --- | --- | --- |
| `GUI_PORT` | 18080 | `/` |
| `API_PORT` | 18081 | `/api` |
| `MCP_PORT` | 18082 | `/mcp` |

Set `BASE_URL` and `CORS_ORIGINS` to the URL browsers and OAuth callbacks use
(for example `http://your-host:18080`).

## Bring your own Compose

Copy this into your own Compose file or merge the services into an existing
stack. Adjust image tag, ports, passwords, and `BASE_URL` / `CORS_ORIGINS`.

```yaml
services:
  memchentic-db:
    image: pgvector/pgvector:pg16
    environment:
      POSTGRES_USER: memchentic
      POSTGRES_PASSWORD: replace-me-db-password
      POSTGRES_DB: memchentic
    volumes:
      - memchentic-db-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U memchentic -d memchentic"]
      interval: 5s
      timeout: 5s
      retries: 10

  memchentic:
    image: ghcr.io/memchentic/memchentic:latest
    depends_on:
      memchentic-db:
        condition: service_healthy
    ports:
      - "18080:8080"   # GUI /
      - "18081:8080"   # API /api
      - "18082:8080"   # MCP /mcp
    environment:
      BASE_URL: http://your-host:18080
      CORS_ORIGINS: http://your-host:18080
      OAUTH_PROVIDER: internal
      ADMIN_EMAIL: admin@example.com
      ADMIN_DISPLAY_NAME: Admin
      ADMIN_PASSWORD: replace-me-admin-password
      ADMIN_API_KEY: replace-me-admin-api-key
      POSTGRES_USER: memchentic
      POSTGRES_PASSWORD: replace-me-db-password
      POSTGRES_HOSTNAME: memchentic-db
      POSTGRES_PORT: "5432"
      POSTGRES_DB: memchentic

volumes:
  memchentic-db-data:
```

You can publish a single host port if a reverse proxy terminates TLS and
routes `/`, `/api`, and `/mcp` to the same upstream.

## Existing Postgres

Skip the database service. Set `POSTGRES_HOSTNAME` (and user / password / db)
to your existing pgvector instance. The application expects a Postgres that
can load the `vector` extension.

Full variable reference: {doc}`environment-variables`. For Kubernetes, see
{doc}`helm`.
