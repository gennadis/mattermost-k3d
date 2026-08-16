# mattermost-k3d
Mattermost deployed on Kubernetes (k3d) from raw manifests

## Stack
- **Postgres** - primary datastore
- **Redis** - deployed and wired, but inactive (see note below)
- **Mattermost** - Go backend + web app ([`mattermost/mattermost`](https://github.com/mattermost/mattermost))

## Cluster
Local [k3d](https://k3d.io) cluster:
```
k3d cluster create learn --servers 1 --agents 1
```

## Deploy
```
kubectl apply -f manifests/namespace.yaml
kubectl apply -f manifests/postgres/
kubectl apply -f manifests/redis/
kubectl apply -f manifests/mattermost/
```

## Access
```
kubectl -n mattermost-k3d port-forward svc/mattermost 8065:8065
```
Then open http://localhost:8065

## Notes
- **Image:** the official `mattermost-team-edition` image has no `arm64` build, so this
  runs `ghcr.io/jannegpriv/mattermost-arm64` instead (Apple Silicon compatible)
- **Redis:** Mattermost's Redis cache backend requires an Enterprise license -
  without one the server refuses to boot. Redis stays deployed to demonstrate the
  integration, but `MM_CACHESETTINGS_CACHETYPE` is left disabled in
  `manifests/mattermost/configmap.yaml`
