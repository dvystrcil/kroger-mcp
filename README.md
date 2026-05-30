# kroger-mcp

Deploy repo for [`dvystrcil/kroger-mcp-docker`](https://github.com/dvystrcil/kroger-mcp-docker) — kustomize base + image-updater CR. Half of the canonical three-repo split for homelab MCPs:

| Repo | Purpose |
|---|---|
| [`dvystrcil/kroger-mcp-docker`](https://github.com/dvystrcil/kroger-mcp-docker) | Source + Dockerfile + CI |
| [`dvystrcil/kroger-mcp`](https://github.com/dvystrcil/kroger-mcp) **(this repo)** | Kustomize base + IU CR |
| [`dvystrcil/argocd-projects`](https://github.com/dvystrcil/argocd-projects) | Argo Application + IU enrollment |

## Layout

```
base/
  namespace.yaml
  infisical-secret.yaml
  harbor-pull-secret.yaml
  image-updater-rbac.yaml
  pvc.yaml
  deployment.yaml
  service.yaml
  kustomization.yaml          ← image override pinned here; IU writes back

image-updater/
  kroger-mcp-image-updater.yaml  ← IU CR, enrolled via image-updater-apps ApplicationSet
```

## Image flow

1. Code merge to `dvystrcil/kroger-mcp-docker` → CI builds image, `dvystrcil/release-action@v0.1.1` auto patch-bumps a GH release (e.g. `v0.0.7`).
2. Release publish fires `docker-release.yaml` in -docker, which retags Harbor `:dev` → `:0.0.7`.
3. argocd-image-updater (via the CR in `image-updater/`) detects the new Harbor tag matching `:0.x` and writes `newTag: 0.0.7` into `base/kustomization.yaml` on this repo's main branch.
4. ArgoCD reconciles, rolls the Deployment.

## License

[MIT](LICENSE).
