# strapi

Self-hosted Strapi CMS with SQLite on a PersistentVolumeClaim and S3 media storage.

Email is sent via SMTP (Domeneshop) using the `@strapi/provider-email-nodemailer` plugin. From address: `balve@garmeres.com`, reply-to: `admin@garmeres.com`.

See [09-create-object-storage.md](../../docs/02-infrastructure/09-create-object-storage.md) for prerequisite secrets and bucket setup.

## Backups

The SQLite database is backed up to S3 every other day (odd days) at 23:00 UTC. The 14 newest backups are retained.

```
s3://balve-strapi/backups/strapi-backup-YYYYMMDD.db.gz
```

## Restore

Scale down Strapi first, then trigger the restore job:

```bash
# Scale down
kubectl scale deployment strapi -n strapi --replicas=0

# Restore latest backup
kubectl create job --from=cronjob/strapi-restore restore-now -n strapi

# Or restore a specific date (YYYYMMDD).
# Available dates can be found in the Hetzner Object Storage console
# under balve-strapi/backups/, or in the restore job logs.
kubectl create job --from=cronjob/strapi-restore restore-now -n strapi \
  --dry-run=client -o json | \
  jq '.spec.template.spec.containers[0].env[0].value = "20260317"' | \
  kubectl apply -f -

# Check job logs to see available backups and progress
kubectl logs -n strapi job/restore-now

# Scale back up
kubectl scale deployment strapi -n strapi --replicas=1

# Clean up the job
kubectl delete job restore-now -n strapi
```
