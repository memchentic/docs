# memchentic docs

User-facing Sphinx (MyST) documentation for
[memchentic](https://github.com/memchentic/memchentic).

Published image: `ghcr.io/memchentic/docs:<tag>` (nginx serving built HTML).

## Layout

| Path | Role |
| --- | --- |
| `sphinx/` | Sphinx project (`source/`, requirements) |
| `assets/` | Brand marks used by the product README |
| `docker/` | Image Dockerfile + nginx config |
| `.github/workflows/publish-docs-image.yml` | Manual multi-arch GHCR publish |

## Local preview

```bash
docker build -t memchentic-docs:local -f docker/Dockerfile .
docker run --rm -p 18099:80 memchentic-docs:local
```

Or from the main memchentic checkout, use the Compose overlay `docker/docs.yaml`
(pulls `DOCS_IMAGE`, or rebuilds from this tree when you pass `--build`).

## Publish

GitHub Actions → **Publish Docs Image** → choose runner, platforms, and
Major / Minor / Patch.
