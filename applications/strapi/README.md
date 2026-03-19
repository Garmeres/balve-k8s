# strapi

Self-hosted Strapi CMS with SQLite on a PersistentVolumeClaim and S3 media storage.

Email is sent via SMTP (Domeneshop) using the `@strapi/provider-email-nodemailer` plugin. From address: `balve@garmeres.com`, reply-to: `admin@garmeres.com`.

See [08-create-object-storage.md](../../docs/day-0/08-create-object-storage.md) for prerequisite secrets and bucket setup.

## Backups

The SQLite database is backed up to S3 every other day (odd days) at 23:00 UTC. Backups are retained for 28 days (~14 backups). Backup files are stored at `s3://balve-strapi/backups/strapi-YYYYMMDD.db.gz`.

## Restore

Scale down Strapi first, then trigger the restore job:

```bash
# Scale down
kubectl scale deployment strapi -n strapi --replicas=0

# Restore latest backup
kubectl create job --from=cronjob/strapi-db-restore restore-now -n strapi

# Or restore a specific backup
kubectl create job --from=cronjob/strapi-db-restore restore-now -n strapi \
  --dry-run=client -o json | \
  jq '.spec.template.spec.containers[0].env[0].value = "strapi-20260317.db.gz"' | \
  kubectl apply -f -

# Check job logs to see available backups and progress
kubectl logs -n strapi job/restore-now

# Scale back up
kubectl scale deployment strapi -n strapi --replicas=1

# Clean up the job
kubectl delete job restore-now -n strapi
```
