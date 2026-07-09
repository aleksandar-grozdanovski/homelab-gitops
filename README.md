# homelab-gitops

GitOps repository for my homelab's single-node K3s cluster (**dellbox**). [Flux CD](https://fluxcd.io/) watches this repo — **a `git push` here is the only way anything changes on the cluster.** No manual `kubectl apply`.

## How it works

1. Flux runs in the cluster and polls this repository (`GitRepository` + `Kustomization` in `clusters/dellbox/`)
2. On every push, Flux compares the manifests here against the live cluster state
3. Anything that differs gets reconciled — new apps deployed, changed configs applied, manual drift reverted

```
git push → Flux detects change → reconcile → cluster converges to this repo
```

## What's deployed

| App | Path | What it is |
|-----|------|------------|
| **RAG API** | `clusters/dellbox/apps/api/` | My [homelab-rag-api](https://github.com/aleksandar-grozdanovski/homelab-rag-api) (.NET 10) + PostgreSQL/pgvector — namespace, secret, PVC, deployments, services |
| **Monitoring** | `clusters/dellbox/apps/monitoring/` | Prometheus + Grafana via `HelmRepository` + `HelmRelease` |

## Layout

```
clusters/dellbox/
├── flux-system-gitrepository.yaml    # Flux: which repo to watch
├── flux-system-kustomization.yaml    # Flux: what to apply from it
└── apps/
    ├── api/                          # RAG API + PostgreSQL (00-06, ordered)
    └── monitoring/                   # kube-prometheus-stack via Helm
```

## Related repos

- [homelab-ansible](https://github.com/aleksandar-grozdanovski/homelab-ansible) — provisions the server itself (Debian, Docker, K3s) from scratch
- [homelab-rag-api](https://github.com/aleksandar-grozdanovski/homelab-rag-api) — the application deployed by this repo
