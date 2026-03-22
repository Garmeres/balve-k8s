# strapi

Self-hosted Strapi CMS with SQLite on a PersistentVolumeClaim and S3 media storage.

Email is sent via SMTP (Domeneshop) using the `@strapi/provider-email-nodemailer` plugin. From address: `balve@garmeres.com`, reply-to: `admin@garmeres.com`.

See [08-create-object-storage.md](../../docs/02-balve-infrastructure/08-create-object-storage.md) for prerequisite secrets and bucket setup.

## Backups

The SQLite database is backed up to S3 every other day (odd days) at 23:00 UTC. The 14 newest backups are retained.

```
s3://balve-strapi/backups/strapi-backup-YYYYMMDD.db.gz
```

## Restore

The procedure below works for both a simple restore and full disaster recovery (PVC deleted).

Trigger an ArgoCD sync to ensure all resources exist (no-op if nothing was deleted):

```bash
kubectl patch app strapi -n argocd --type merge \
  -p '{"operation":{"initiatedBy":{"username":"admin"},"sync":{}}}'
```

Wait for the PVC to be ready:

```bash
kubectl wait --for=jsonpath=.status.phase=Bound pvc/strapi-data -n strapi --timeout=60s
```

Pause ArgoCD auto-sync so scale-down is not reverted:

```bash
kubectl patch app strapi -n argocd --type merge \
  -p '{"spec":{"syncPolicy":null}}'
```

Scale down Strapi:

```bash
kubectl scale deployment strapi -n strapi --replicas=0
```

Restore the latest backup:

```bash
kubectl create job --from=cronjob/strapi-restore restore-strapi -n strapi
```

Or restore a specific date (YYYYMMDD). Available dates are listed in the job logs, or in the Hetzner Object Storage console under `balve-strapi/backups/`:

```bash
kubectl create job --from=cronjob/strapi-restore restore-strapi -n strapi \
  --dry-run=client -o json | \
  jq '(.spec.template.spec.containers[0].env[] | select(.name == "RESTORE_DATE")).value = "20260317"' | \
  kubectl apply -f -
```

Follow the job logs (waits up to 60 s for the pod to start):

```bash
kubectl logs -f --pod-running-timeout=60s -n strapi job/restore-strapi
```

Scale back up:

```bash
kubectl scale deployment strapi -n strapi --replicas=1
```

Re-enable ArgoCD auto-sync:

```bash
kubectl patch app strapi -n argocd --type merge \
  -p '{"spec":{"syncPolicy":{"automated":{"prune":true,"selfHeal":true}}}}'
```

Clean up the job:

```bash
kubectl delete job restore-strapi -n strapi
```
