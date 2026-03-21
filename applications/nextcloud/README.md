# Nextcloud

Nextcloud 33 with MariaDB, Redis, local disk primary storage, and self-hosted Collabora Online for document editing.

See [08-create-object-storage.md](../../docs/02-infrastructure/08-create-object-storage.md) for prerequisite secrets and bucket setup.

## Installed apps

The following apps are installed automatically on pod startup:

| App           | Purpose                                              |
| ------------- | ---------------------------------------------------- |
| calendar      | Calendar                                             |
| external      | Embed external sites (ArgoCD, Strapi) as iframes     |
| forms         | Forms and surveys                                    |
| oidc          | OpenID Connect — allows ArgoCD to authenticate users |
| passwords     | Password manager                                     |
| polls         | Polls                                                |
| richdocuments | Collabora Online integration                         |
| side_menu     | Sidebar navigation menu                              |

Also installed: drop_account, group_everyone, impersonate, suspicious_login, twofactor_admin, twofactor_nextcloud_notification, twofactor_totp, twofactor_webauthn.

## Collabora Online

Collabora Online is deployed as a subchart and provides in-browser document editing. It runs at [collabora.balve.garmeres.com](https://collabora.balve.garmeres.com) and connects to Nextcloud via WOPI.

The WOPI URLs are set on each pod start via `postStartCommand` in `values.yaml`.

## Nightly schedule (all times UTC)

| Time        | Job                | Schedule             |
| ----------- | ------------------ | -------------------- |
| 23:00       | Backup             | Even days (`2-30/2`) |
| 00:00       | Image updater      | Daily                |
| 01:00–05:00 | Maintenance window | Daily                |

## Image updates

The `nextcloud-image-updater` CronJob runs daily at midnight UTC. It compares the running image digest against the remote `33-apache` tag on Docker Hub. If a new patch image is available, it triggers a rolling restart. Major version upgrades (e.g. 33 → 34) require manually changing `image.tag` in `values.yaml`.

## Backups

The MariaDB database, `config.php`, and all user files are backed up to S3 every other day (even days) at 23:00 UTC. Backups are retained for 28 days (~14 backups).

Each backup is a single tar.gz archive:

```
s3://<bucket>/backups/nextcloud-backup-YYYYMMDD.tar.gz
├── nextcloud.sql    # MariaDB database dump
├── config.php       # instanceid, secret, passwordsalt
└── userdata.tar.gz  # all user files from /data
```

## Restore

The restore job needs MariaDB running and the Nextcloud PVC bound. The procedure below works for both a simple restore and full disaster recovery (PVCs deleted).

Trigger an ArgoCD sync to ensure all resources exist (no-op if nothing was deleted):

```bash
kubectl patch app nextcloud -n argocd --type merge \
  -p '{"operation":{"initiatedBy":{"username":"admin"},"sync":{}}}'
```

Wait for PVCs and MariaDB to be ready:

```bash
kubectl wait --for=jsonpath=.status.phase=Bound pvc/nextcloud-nextcloud \
  pvc/data-nextcloud-mariadb-0 -n nextcloud --timeout=60s
kubectl rollout status statefulset/nextcloud-mariadb -n nextcloud --timeout=120s
```

Pause ArgoCD auto-sync so scale-down is not reverted:

```bash
kubectl patch app nextcloud -n argocd --type merge \
  -p '{"spec":{"syncPolicy":null}}'
```

Scale down Nextcloud (MariaDB stays running for the database import):

```bash
kubectl scale deployment nextcloud -n nextcloud --replicas=0
```

Restore the latest backup:

```bash
kubectl create job --from=cronjob/nextcloud-restore restore-nextcloud -n nextcloud
```

Or restore a specific date (YYYYMMDD). Available dates are listed in the job logs, or in the Hetzner Object Storage console under `balve-nextcloud/backups/`:

```bash
kubectl create job --from=cronjob/nextcloud-restore restore-nextcloud -n nextcloud \
  --dry-run=client -o json | \
  jq '(.spec.template.spec.containers[0].env[] | select(.name == "RESTORE_DATE")).value = "20260317"' | \
  kubectl apply -f -
```

Follow the job logs (waits up to 60 s for the pod to start):

```bash
kubectl logs -f --pod-running-timeout=60s -n nextcloud job/restore-nextcloud
```

Scale up and restart to reinstall apps via `postStartCommand`:

```bash
kubectl scale deployment nextcloud -n nextcloud --replicas=1
kubectl rollout restart deployment/nextcloud -n nextcloud
kubectl rollout status deployment/nextcloud -n nextcloud --timeout=300s
```

Re-enable ArgoCD auto-sync:

```bash
kubectl patch app nextcloud -n argocd --type merge \
  -p '{"spec":{"syncPolicy":{"automated":{"prune":true,"selfHeal":true}}}}'
```

Clean up the job:

```bash
kubectl delete job restore-nextcloud -n nextcloud
```

## Sealed secrets

| Secret              | Keys                                                                  |
| ------------------- | --------------------------------------------------------------------- |
| `nextcloud-admin`   | `username`, `password`, `smtp-host`, `smtp-username`, `smtp-password` |
| `nextcloud-s3`      | `S3_ACCESS_KEY_ID`, `S3_SECRET_ACCESS_KEY`                            |
| `nextcloud-mariadb` | `mariadb-root-password`, `mariadb-password`, `db-username`            |
| `nextcloud-redis`   | `redis-password`                                                      |
