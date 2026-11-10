# Docker

Run Memchentic with Docker Compose: the app plus Postgres with vector search.

Image: `registry.memchentic.ai/memchentic/memchentic:latest` (preferred registry).
Version tags are also published; `latest` moves with each release. The same tags are
mirrored to `ghcr.io/memchentic/memchentic`.

## What you need

| Piece | Notes |
| --- | --- |
| Memchentic image | From the registry as above (anonymous pull) |
| Postgres 16 with pgvector | `pgvector/pgvector:pg16` works |
| Secrets | Strong `ADMIN_PASSWORD` and `ADMIN_API_KEY` (16+ characters each) |

The first start prepares the database automatically.

## Ready-made Compose in the repo

The Memchentic repository includes a Compose stack under `docker/`
(`docker-compose.yaml` and `.env.example`). Use that as-is, or copy the shape
into your own project.

From a clone:

```bash
cd docker
cp .env.example .env
# set strong ADMIN_PASSWORD and ADMIN_API_KEY (16+ characters each)
docker compose up -d
```

Compose publishes one host port, `GUI_PORT` (default 18080). The web UI, the
HTTP API (`/api`), and the agent / MCP endpoint (`/mcp`) all share it.

The port binds to `BIND_ADDRESS`, which defaults to `127.0.0.1` (this machine
only). Set `BIND_ADDRESS=0.0.0.0` in `.env` to reach Memchentic from other
machines, or put a reverse proxy in front of it.

Database migrations run when the container starts. To run them as a separate
step instead, set `RUN_MIGRATIONS_ON_BOOT=false` and use the one-shot migrate
overlay:

```bash
COMPOSE_FILE=docker-compose.yaml:migrate.yaml docker compose up -d
```

Set `BASE_URL` and `CORS_ORIGINS` to the URL people open in the browser (for
example `http://your-host:18080`).

## Bring your own Compose

```yaml
services:
  memchentic-db:
    image: pgvector/pgvector:0.8.7-pg16
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
    image: registry.memchentic.ai/memchentic/memchentic:latest
    depends_on:
      memchentic-db:
        condition: service_healthy
    ports:
      - "18080:8080"   # web UI, /api, and /mcp
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

You can expose a single public port instead and let a reverse proxy send web
UI, API, and agent traffic to Memchentic.

## Existing Postgres

Skip the database service. Point `POSTGRES_HOSTNAME` (and user / password /
database name) at a Postgres that supports the `vector` extension.

Full settings list: {doc}`environment-variables`. For Kubernetes, see
{doc}`helm`.
