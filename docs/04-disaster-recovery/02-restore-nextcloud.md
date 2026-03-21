# Restore Nextcloud

Restores Nextcloud from an S3 backup. This includes the database (all users, files, calendar, contacts), the configuration, and user files. Works whether the data is corrupt or was completely deleted.

Backups are created every other day (even days) at 23:00 UTC and kept for 28 days. Each backup is stored in Hetzner Object Storage under `balve-nextcloud/backups/`.

---

## Before you start

You need server access. If you're not already logged in:

1. Log in to [console.hetzner.cloud](https://console.hetzner.cloud) with your own Hetzner account
2. Open the project → _Servers_ → `master-1`
3. Click the **>\_ Console** icon (top right) to open a web terminal
4. Log in as `root`

All commands below run on this server.

---

## Step 1: Make sure the app resources exist

ArgoCD manages all Nextcloud resources (database, storage, etc). Trigger a sync to make sure everything is in place — if nothing was deleted, this does nothing:

```
kubectl patch app nextcloud -n argocd --type merge \
  -p '{"operation":{"initiatedBy":{"username":"admin"},"sync":{}}}'
```

Wait for the storage volumes and database to be ready:

```
kubectl wait --for=jsonpath=.status.phase=Bound pvc/nextcloud-nextcloud \
  pvc/data-nextcloud-mariadb-0 -n nextcloud --timeout=60s
kubectl rollout status statefulset/nextcloud-mariadb -n nextcloud --timeout=120s
```

If this times out, check the ArgoCD dashboard at [argocd.balve.garmeres.com](https://argocd.balve.garmeres.com) — the `nextcloud` app may have an error that needs to be resolved first (e.g. missing secrets → [Re-seal secrets](04-reseal-secrets.md)).

---

## Step 2: Pause ArgoCD

ArgoCD will try to undo the changes you make in the next step (scaling down Nextcloud). Pause it:

```
kubectl patch app nextcloud -n argocd --type merge \
  -p '{"spec":{"syncPolicy":null}}'
```

---

## Step 3: Stop Nextcloud

Stop the Nextcloud application so the restore job can safely replace the data. The database (MariaDB) stays running because the restore job needs it:

```
kubectl scale deployment nextcloud -n nextcloud --replicas=0
kubectl wait --for=delete pod -l app.kubernetes.io/name=nextcloud -n nextcloud --timeout=60s
```

---

## Step 4: Restore from backup

**To restore the latest backup:**

```
kubectl create job --from=cronjob/nextcloud-restore restore-nextcloud -n nextcloud
```

**To restore a specific date**, replace `20260317` with the date you want (YYYYMMDD). You can see available backup dates in the Hetzner Object Storage console under `balve-nextcloud/backups/`:

```
kubectl create job --from=cronjob/nextcloud-restore restore-nextcloud -n nextcloud \
  --dry-run=client -o json | \
  jq '(.spec.template.spec.containers[0].env[] | select(.name == "RESTORE_DATE")).value = "20260317"' | \
  kubectl apply -f -
```

Watch the restore progress (this may take a minute to start):

```
kubectl logs -f --pod-running-timeout=60s -n nextcloud job/restore-nextcloud
```

Wait for the job to complete. If the logs show errors, read the message — it usually explains what went wrong (e.g. backup not found for the given date).

---

## Step 5: Start Nextcloud

Start Nextcloud again and wait for it to become ready:

```
kubectl scale deployment nextcloud -n nextcloud --replicas=1
kubectl rollout restart deployment/nextcloud -n nextcloud
kubectl rollout status deployment/nextcloud -n nextcloud --timeout=300s
```

---

## Step 6: Re-enable ArgoCD

Turn auto-sync back on so ArgoCD manages Nextcloud again:

```
kubectl patch app nextcloud -n argocd --type merge \
  -p '{"spec":{"syncPolicy":{"automated":{"prune":true,"selfHeal":true}}}}'
```

---

## Step 7: Clean up

Delete the restore job (it's no longer needed):

```
kubectl delete job restore-nextcloud -n nextcloud
```

---

## Step 8: Verify

Open [https://balve.garmeres.com](https://balve.garmeres.com) and confirm files, calendar, and contacts are present.

---

## Admin credentials

You normally don't need admin access — all users log in with their own accounts. But if you need to manage users or change settings, you can reset any user's password from the server without knowing the current one:

```
kubectl exec -it deploy/nextcloud -n nextcloud -- su -s /bin/bash www-data -c "php occ user:resetpassword <username>"
```

If you don't know which user has admin privileges:

```
kubectl exec deploy/nextcloud -n nextcloud -- su -s /bin/bash www-data -c "php occ group:list --output=json" | jq -r '.admin[]'
```
