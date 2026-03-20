# Nextcloud

Nextcloud 33 with MariaDB, Redis, S3 primary objectstore, and self-hosted Collabora Online for document editing.

See [09-create-object-storage.md](../../docs/02-infrastructure/09-create-object-storage.md) for prerequisite secrets and bucket setup.

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

The MariaDB database and `config.php` are backed up to S3 every other day (even days) at 23:00 UTC. Backups are retained for 28 days (~14 backups).

Each backup is a single tar.gz archive containing both the database dump and config.php:

```
s3://<bucket>/backups/nextcloud-backup-YYYYMMDD.tar.gz
├── nextcloud.sql    # MariaDB database dump
└── config.php       # instanceid, secret, passwordsalt
```

## Restore

ArgoCD auto-sync must be paused first, otherwise it will scale the deployment back up immediately.

```bash
# Pause ArgoCD auto-sync
kubectl patch app nextcloud -n argocd --type merge \
  -p '{"spec":{"syncPolicy":null}}'

# Scale down
kubectl scale deployment nextcloud -n nextcloud --replicas=0

# Restore latest backup
kubectl create job --from=cronjob/nextcloud-restore restore-now -n nextcloud

# Or restore a specific date (YYYYMMDD).
# Available dates can be found in the Hetzner Object Storage console
# under balve-nextcloud/backups/, or in the restore job logs.
kubectl create job --from=cronjob/nextcloud-restore restore-now -n nextcloud \
  --dry-run=client -o json | \
  jq '.spec.template.spec.containers[0].env[0].value = "20260317"' | \
  kubectl apply -f -

# Check job logs to see available backups and progress
kubectl logs -n nextcloud job/restore-now

# Scale back up
kubectl scale deployment nextcloud -n nextcloud --replicas=1

# Re-enable ArgoCD auto-sync
kubectl patch app nextcloud -n argocd --type merge \
  -p '{"spec":{"syncPolicy":{"automated":{"prune":true,"selfHeal":true}}}}'

# Clean up the job
kubectl delete job restore-now -n nextcloud
```

## Sealed secrets

| Secret              | Keys                                                                  |
| ------------------- | --------------------------------------------------------------------- |
| `nextcloud-admin`   | `username`, `password`, `smtp-host`, `smtp-username`, `smtp-password` |
| `nextcloud-s3`      | `S3_ACCESS_KEY_ID`, `S3_SECRET_ACCESS_KEY`                            |
| `nextcloud-mariadb` | `mariadb-root-password`, `mariadb-password`, `db-username`            |
| `nextcloud-redis`   | `redis-password`                                                      |
