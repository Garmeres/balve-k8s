# Nextcloud

Nextcloud 33 with MariaDB, Redis, S3 primary objectstore, and Collabora Online.

See [08-create-object-storage.md](../../docs/day-0/08-create-object-storage.md) for prerequisite secrets and bucket setup.

## Nightly schedule (all times UTC)

| Time  | Job                  | Schedule               |
| ----- | -------------------- | ---------------------- |
| 23:00 | Database backup      | Even days (`2-30/2`)   |
| 00:00 | Image updater        | Daily                  |
| 01:00–05:00 | Maintenance window | Daily (set via config) |

## Image updates

The `nextcloud-image-updater` CronJob runs daily at midnight UTC. It compares the running image digest against the remote `33-apache` tag on Docker Hub. If a new patch image is available, it triggers a rolling restart. Major version upgrades (e.g. 33 → 34) require manually changing `image.tag` in `values.yaml`.

## Backups

The MariaDB database is backed up to S3 every other day (even days) at 23:00 UTC. Backups are retained for 28 days (~14 backups). Backup files are stored at `s3://<bucket>/backups/nextcloud-YYYYMMDD.sql.gz`.

## Restore

Scale down Nextcloud first, then trigger the restore job:

```bash
# Scale down
kubectl scale deployment nextcloud -n nextcloud --replicas=0

# Restore latest backup
kubectl create job --from=cronjob/nextcloud-db-restore restore-now -n nextcloud

# Or restore a specific backup
kubectl create job --from=cronjob/nextcloud-db-restore restore-now -n nextcloud \
  --dry-run=client -o json | \
  jq '.spec.template.spec.containers[0].env[0].value = "nextcloud-20260317.sql.gz"' | \
  kubectl apply -f -

# Check job logs to see available backups and progress
kubectl logs -n nextcloud job/restore-now

# Scale back up
kubectl scale deployment nextcloud -n nextcloud --replicas=1

# Clean up the job
kubectl delete job restore-now -n nextcloud
```

## Sealed secrets

| Secret             | Keys                                                        |
| ------------------ | ----------------------------------------------------------- |
| `nextcloud-admin`  | `username`, `password`, `smtp-host`, `smtp-username`, `smtp-password` |
| `nextcloud-s3`     | `S3_ACCESS_KEY_ID`, `S3_SECRET_ACCESS_KEY`                  |
| `nextcloud-mariadb`| `mariadb-root-password`, `mariadb-password`, `db-username`  |
| `nextcloud-redis`  | `redis-password`                                            |
