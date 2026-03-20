# Restore Strapi

Restores the Strapi SQLite database from an S3 backup. Works whether the data is corrupt or the PVC was deleted.

All commands run on the server. To access it:

1. Go to [console.hetzner.cloud](https://console.hetzner.cloud) and log in
2. Open the project → _Servers_ → `master-1`
3. Click the **>\_ Console** icon (top right) to open a web terminal
4. Log in as `root`

---

## Ensure resources exist

Trigger an ArgoCD sync (no-op if nothing was deleted):

```
kubectl patch app strapi -n argocd --type merge \
  -p '{"operation":{"initiatedBy":{"username":"admin"},"sync":{}}}'
```

Wait for the PVC to be ready:

```
kubectl wait --for=jsonpath=.status.phase=Bound pvc/strapi-data -n strapi --timeout=60s
```

## Pause ArgoCD

Pause auto-sync so scale-down is not reverted:

```
kubectl patch app strapi -n argocd --type merge \
  -p '{"spec":{"syncPolicy":null}}'
```

## Scale down Strapi

```
kubectl scale deployment strapi -n strapi --replicas=0
```

## Restore

Restore the latest backup:

```
kubectl create job --from=cronjob/strapi-restore restore-strapi -n strapi
```

Or restore a specific date (YYYYMMDD). Available dates are listed in the job logs, or in the Hetzner Object Storage console under `balve-strapi/backups/`:

```
kubectl create job --from=cronjob/strapi-restore restore-strapi -n strapi \
  --dry-run=client -o json | \
  jq '(.spec.template.spec.containers[0].env[] | select(.name == "RESTORE_DATE")).value = "20260317"' | \
  kubectl apply -f -
```

Follow the job logs:

```
kubectl logs -f --pod-running-timeout=60s -n strapi job/restore-strapi
```

## Scale up

```
kubectl scale deployment strapi -n strapi --replicas=1
```

## Re-enable ArgoCD

```
kubectl patch app strapi -n argocd --type merge \
  -p '{"spec":{"syncPolicy":{"automated":{"prune":true,"selfHeal":true}}}}'
```

## Clean up

```
kubectl delete job restore-strapi -n strapi
```

## Verify

Open [https://strapi.balve.garmeres.com/admin](https://strapi.balve.garmeres.com/admin) and confirm blog posts and pages are present.

## Admin credentials

The Strapi admin credentials are normally not needed — they are only required for managing content types and users in the admin panel. If you need to recover access:

1. Go to [strapi.balve.garmeres.com/admin](https://strapi.balve.garmeres.com/admin)
2. Click **Forgot your password?**
3. Enter `admin@garmeres.com`
4. The reset email is sent to the `admin@garmeres.com` email group
