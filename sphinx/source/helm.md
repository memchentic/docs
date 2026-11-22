# Helm

Install Memchentic on Kubernetes with the chart in this repository. The chart
can run its own Postgres with vector search, or point at a database you
already have.

Default image: `registry.memchentic.ai/memchentic/memchentic:latest` (preferred; mirrored to GHCR)

## Install

```bash
helm upgrade --install memchentic ./helm \
  -n memchentic --create-namespace \
  -f my-values.yaml
```

Set `config.baseUrl` and `config.corsOrigins` to the public URL people open in
the browser. The Service listens on port **8080**. Ingress is off by default.

## Postgres

| Mode | Values |
| --- | --- |
| In-cluster | `postgres.enabled: true` (default) — includes storage |
| External | `postgres.enabled: false` and `config.postgresHostname: my.db.example` |

The database password always comes from the resolved Secret.

The in-cluster Postgres runs as the image's `postgres` user (uid/gid 999) with
no extra privileges. If your storage ignores `fsGroup`, override
`postgres.podSecurityContext`.

A NetworkPolicy lets only the Memchentic pods reach the in-cluster Postgres
(`postgres.networkPolicy.enabled: true`). It only has an effect when your
cluster's network plugin enforces NetworkPolicy. List any other workload that
must connect, such as a backup job, under `postgres.networkPolicy.extraFrom`.

## Secret modes

Set `secrets.mode` to exactly one of:

| Mode | Behavior |
| --- | --- |
| `inline` | Chart creates a Secret from `secrets.inline.*` (fine for quick trials). |
| `existing` | Use `secrets.existingSecret` (any secret sync you already run). |
| `externalSecret` | Chart creates an ExternalSecret; your cluster needs External Secrets Operator and a SecretStore. |

Required Secret keys:

- `ADMIN_PASSWORD`
- `ADMIN_API_KEY`
- `POSTGRES_PASSWORD`
- `OAUTH_CLIENT_SECRET` (may be empty when using built-in email/password login)

Admin password and API key must be at least 16 characters.

An empty or missing `secrets.mode` means `existing`.

### External Secrets example

Vault (KV v2 path example `secret/k8s/memchentic`):

```yaml
secrets:
  mode: externalSecret
  externalSecret:
    secretStoreRef:
      name: memchentic-vault
      kind: SecretStore
    data:
      ADMIN_PASSWORD:
        key: k8s/memchentic
        property: ADMIN_PASSWORD
      ADMIN_API_KEY:
        key: k8s/memchentic
        property: ADMIN_API_KEY
      POSTGRES_PASSWORD:
        key: k8s/memchentic
        property: POSTGRES_PASSWORD
      OAUTH_CLIENT_SECRET:
        key: k8s/memchentic
        property: OAUTH_CLIENT_SECRET
```

AWS Secrets Manager and 1Password use the same idea with their own provider
fields. See `helm/README.md` in the repository for more.

## Replicas, probes and service account

`replicaCount` defaults to `1`. Multi-replica is supported: auth rate limits,
bootstrap and hygiene automation coordinate through Postgres. Size
`config.dbPoolSize` and `config.dbMaxOverflow` so

```text
replicas × (pool_size + max_overflow + 1 LISTEN) < postgres max_connections
```

with headroom. See `helm/examples/scaled.yaml` in the application repository.
When running more than one replica, prefer
`migrations.initContainer.enabled: true` so app pods do not hold the
owner-role password. Boot DDL and Alembic still use a Postgres advisory lock
across rolling updates.

For an external or CloudNativePG database, see
`helm/examples/external-cnpg.yaml`. The owner role (`POSTGRES_USER`) must be
a Postgres superuser.

Liveness and startup probes use `/api/health`. The readiness probe path is
`probes.readinessPath` (default `/api/ready`). The pod then leaves the
Service while the database or embedder is not ready, and it is not restarted.

The chart creates a ServiceAccount only with `serviceAccount.create: true`.
The application never calls the Kubernetes API, so the token is not mounted
unless you set `serviceAccount.automount: true`.

## Upgrading from chart 0.1.x

Chart 0.2.0 changes the application's selector labels, and Kubernetes cannot
change a Deployment selector in place. Delete the Deployment once, then
upgrade. The database and its volume are not touched.

```bash
kubectl delete deployment <release>-memchentic   # just <release> when it contains "memchentic"
helm upgrade <release> ./helm -f my-values.yaml
```

Object names are now cut to 43 characters. A release with a very long name
gets shorter object names after the upgrade.

## Useful settings

`config.*` covers the public URL, login provider (default built-in), admin
email, database name, search defaults, session lifetimes, CORS, log level,
and optional outbound mail (`config.smtpHost`, `config.smtpPort`,
`config.smtpUsername`, `config.smtpFrom`, `config.smtpUseTls`). Put
`SMTP_PASSWORD` in the chart secret (inline, existing Secret, or
ExternalSecret). Empty host or From leaves mail off; afterwards admins edit
mail under System Settings. Details: {doc}`environment-variables`.

Prefer Compose instead? See {doc}`docker`.
