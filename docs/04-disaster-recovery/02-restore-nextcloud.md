# Restore Nextcloud

Restores Nextcloud from an S3 backup. This includes the database (all users, files, calendar, contacts), the configuration, and user files. Works whether the data is corrupt or was completely deleted.

Backups are created every other day (even days) at 23:00 UTC and kept for 28 days. Each backup is stored in Hetzner Object Storage under `balve-nextcloud/backups/`.

---

## Before you start

You need `ssh balve-master` to work. If it doesn't, set it up first — see [Set up SSH access](01-triage.md#set-up-ssh-access).

All commands below run on your local machine.

---

## Step 1: Make sure the app resources exist

ArgoCD manages all Nextcloud resources (database, storage, etc). Trigger a sync to make sure everything is in place — if nothing was deleted, this does nothing:

```
ssh balve-master "kubectl patch app nextcloud -n argocd --type merge -p '{\"operation\":{\"initiatedBy\":{\"username\":\"admin\"},\"sync\":{}}}'"
```

Wait for the storage volumes and database to be ready:

```
ssh balve-master "kubectl wait --for=jsonpath=.status.phase=Bound pvc/nextcloud-nextcloud pvc/data-nextcloud-mariadb-0 -n nextcloud --timeout=60s"
ssh balve-master "kubectl rollout status statefulset/nextcloud-mariadb -n nextcloud --timeout=120s"
```

If this times out, check the ArgoCD dashboard at [argocd.balve.garmeres.com](https://argocd.balve.garmeres.com) — the `nextcloud` app may have an error that needs to be resolved first (e.g. missing secrets → [Re-seal secrets](04-reseal-secrets.md)).

---

## Step 2: Pause ArgoCD

ArgoCD will try to undo the changes you make in the next step (scaling down Nextcloud). Pause it:

```
ssh balve-master "kubectl patch app nextcloud -n argocd --type merge -p '{\"spec\":{\"syncPolicy\":null}}'"
```

---

## Step 3: Stop Nextcloud

Stop the Nextcloud application so the restore job can safely replace the data. The database (MariaDB) stays running because the restore job needs it:

```
ssh balve-master "kubectl scale deployment nextcloud -n nextcloud --replicas=0"
ssh balve-master "kubectl wait --for=delete pod -l app.kubernetes.io/name=nextcloud -n nextcloud --timeout=60s"
```

---

## Step 4: Restore from backup

**To restore the latest backup:**

```
ssh balve-master "kubectl create job --from=cronjob/nextcloud-restore restore-nextcloud -n nextcloud"
```

**To restore a specific date**, replace `20260317` with the date you want (YYYYMMDD). You can see available backup dates in the Hetzner Object Storage console under `balve-nextcloud/backups/`:

```
ssh balve-master "kubectl create job --from=cronjob/nextcloud-restore restore-nextcloud -n nextcloud --dry-run=client -o json | jq '(.spec.template.spec.containers[0].env[] | select(.name == \"RESTORE_DATE\")).value = \"20260317\"' | kubectl apply -f -"
```

Watch the restore progress (the pod may take a minute or two to start):

```
ssh balve-master "kubectl logs -f --pod-running-timeout=120s -n nextcloud job/restore-nextcloud"
```

If it fails with `is waiting to start: ContainerCreating`, wait a moment and re-run the command.

Wait for the job to complete. If the logs show errors, read the message — it usually explains what went wrong (e.g. backup not found for the given date).

---

## Step 5: Start Nextcloud

Start Nextcloud again and wait for it to become ready:

```
ssh balve-master "kubectl scale deployment nextcloud -n nextcloud --replicas=1"
ssh balve-master "kubectl rollout restart deployment/nextcloud -n nextcloud"
ssh balve-master "kubectl rollout status deployment/nextcloud -n nextcloud --timeout=300s"
```

---

## Step 6: Re-enable ArgoCD

Turn auto-sync back on so ArgoCD manages Nextcloud again:

```
ssh balve-master "kubectl patch app nextcloud -n argocd --type merge -p '{\"spec\":{\"syncPolicy\":{\"automated\":{\"prune\":true,\"selfHeal\":true}}}}'"
```

---

## Step 7: Clean up

Delete the restore job (it's no longer needed):

```
ssh balve-master "kubectl delete job restore-nextcloud -n nextcloud"
```

---

## Step 8: Verify ArgoCD login

The restore job automatically re-registers the ArgoCD OAuth 2.0 client (if the `argocd-oauth` sealed secret exists in the nextcloud namespace — see [03-nextcloud-oidc.md](../03-balve-deployment/03-nextcloud-oidc.md)).

Restart Dex so it reconnects to the restored Nextcloud:

```
ssh balve-master "kubectl rollout restart deployment argocd-dex-server -n argocd"
```

Open [https://argocd.balve.garmeres.com](https://argocd.balve.garmeres.com) and verify you can log in with Nextcloud.

---

## Step 9: Verify

Open [https://balve.garmeres.com](https://balve.garmeres.com) and confirm files, calendar, and contacts are present.

---

## Admin credentials

You normally don't need admin access — all users log in with their own accounts. But if you need to manage users or change settings, you can reset any user's password from the server without knowing the current one:

```
ssh -t balve-master "kubectl exec -it deploy/nextcloud -n nextcloud -- su -s /bin/bash www-data -c 'php occ user:resetpassword <username>'"
```

If you don't know which user has admin privileges:

```
ssh balve-master "kubectl exec deploy/nextcloud -n nextcloud -- su -s /bin/bash www-data -c 'php occ group:list --output=json'" | jq -r '.admin[]'
```
