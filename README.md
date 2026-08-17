# mattermost-k3d
Mattermost deployed on Kubernetes (k3d) from raw manifests

## Stack
- **Mattermost** - Go backend + web app ([`mattermost/mattermost`](https://github.com/mattermost/mattermost))
- **Postgres** - primary datastore
- **Redis** - deployed and wired, but inactive (see note below)
- **MinIO** - S3-compatible object storage for Mattermost file uploads

## Cluster
Local [k3d](https://k3d.io) cluster, with port 80 published so Ingress traffic reaches it:
```
k3d cluster create learn --servers 1 --agents 1 -p "80:80@loadbalancer"
```

## Deploy
```
kubectl apply -f manifests/namespace.yaml
kubectl apply -f manifests/postgres/
kubectl apply -f manifests/redis/
kubectl apply -f manifests/minio/
kubectl apply -f manifests/mattermost/
kubectl apply -f manifests/network-policies/
kubectl apply -f manifests/backup/
```

## Access
Via `Traefik` ingress, routed through [nip.io](https://nip.io) so no `/etc/hosts` edits are needed:
http://mattermost.127.0.0.1.nip.io

Or without relying on the Ingress:
```
kubectl -n mattermost-k3d port-forward svc/mattermost 8065:8065
```
Then open http://localhost:8065

## TODO
- [ ] Small custom Kubernetes operator
- [ ] Docs polishing

## Notes
- **Image:** the official `mattermost-team-edition` image has no `arm64` build, so this
  runs `ghcr.io/jannegpriv/mattermost-arm64` instead (Apple Silicon compatible)
- **Redis:** Mattermost's Redis cache backend requires an Enterprise license -
  without one the server refuses to boot. Redis stays deployed to demonstrate the
  integration, but `MM_CACHESETTINGS_CACHETYPE` is left disabled in
  `manifests/mattermost/configmap.yaml`
- **Network policies:** `manifests/network-policies/` locks down the namespace to
  default-deny ingress, then explicitly allows only: Mattermost -> Postgres/Redis/MinIO,
  the bucket-creation Job -> MinIO, the backup CronJob -> Postgres/MinIO, and the Traefik
  ingress controller (`kube-system`) -> Mattermost
- **Backups:** `manifests/backup/cronjob.yaml` dumps Postgres nightly at 03:00 and uploads
  it to a `postgres-backups` bucket in MinIO. Trigger a manual run with
  `kubectl -n mattermost-k3d create job --from=cronjob/postgres-backup <name>`
