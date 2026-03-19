# Nextcloud

Nextcloud with MariaDB and S3 primary objectstore.

See [08-create-object-storage.md](../../docs/day-0/08-create-object-storage.md) for prerequisite secrets and bucket setup.

## Backups

The MariaDB database is backed up to S3 every other day (odd days) at 3:00 AM. Backups are retained for 28 days (~14 backups). Backup files are stored at `s3://<bucket>/backups/nextcloud-YYYYMMDD.sql.gz`.

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
