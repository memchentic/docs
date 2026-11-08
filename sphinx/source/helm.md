# Helm

Install Memchentic on Kubernetes with the chart in this repository. The chart
can run its own Postgres with vector search, or point at a database you
already have.

Default image: `ghcr.io/memchentic/memchentic:latest`

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

## Useful settings

`config.*` covers the public URL, login provider (default built-in), admin
email, database name, search defaults, session lifetimes, CORS, and log level.
Details: {doc}`environment-variables`.

Prefer Compose instead? See {doc}`docker`.
