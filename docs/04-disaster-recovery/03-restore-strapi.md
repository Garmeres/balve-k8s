# Restore Strapi

Restores the Strapi database from an S3 backup. This brings back all blog posts, pages, and content for the [Garmeres website](https://garmeres.com). Works whether the data is corrupt or was completely deleted.

Backups are created every other day (odd days) at 23:00 UTC. The 14 newest backups are kept in Hetzner Object Storage under `balve-strapi/backups/`.

---

## Before you start

You need `ssh balve-master` to work. If it doesn't, set it up first — see [Set up SSH access](01-triage.md#set-up-ssh-access).

All commands below run on your local machine.

---

## Step 1: Make sure the app resources exist

Trigger an ArgoCD sync to make sure everything is in place — if nothing was deleted, this does nothing:

```
ssh balve-master "kubectl patch app strapi -n argocd --type merge -p '{\"operation\":{\"initiatedBy\":{\"username\":\"admin\"},\"sync\":{}}}'"
```

Wait for the storage volume to be ready:

```
ssh balve-master "kubectl wait --for=jsonpath=.status.phase=Bound pvc/strapi-data -n strapi --timeout=60s"
```

If this times out, check the ArgoCD dashboard at [argocd.balve.garmeres.com](https://argocd.balve.garmeres.com) — the `strapi` app may have an error that needs to be resolved first (e.g. missing secrets → [Re-seal secrets](04-reseal-secrets.md)).

---

## Step 2: Pause ArgoCD

ArgoCD will try to undo the changes you make in the next step (scaling down Strapi). Pause it:

```
ssh balve-master "kubectl patch app strapi -n argocd --type merge -p '{\"spec\":{\"syncPolicy\":null}}'"
```

---

## Step 3: Stop Strapi

Stop the application so the restore job can safely replace the database:

```
ssh balve-master "kubectl scale deployment strapi -n strapi --replicas=0"
ssh balve-master "kubectl wait --for=delete pod -l app=strapi -n strapi --timeout=60s"
```

---

## Step 4: Restore from backup

**To restore the latest backup:**

```
ssh balve-master "kubectl create job --from=cronjob/strapi-restore restore-strapi -n strapi"
```

**To restore a specific date**, replace `20260317` with the date you want (YYYYMMDD). You can see available backup dates in the Hetzner Object Storage console under `balve-strapi/backups/`:

```
ssh balve-master "kubectl create job --from=cronjob/strapi-restore restore-strapi -n strapi --dry-run=client -o json | jq '(.spec.template.spec.containers[0].env[] | select(.name == \"RESTORE_DATE\")).value = \"20260317\"' | kubectl apply -f -"
```

Watch the restore progress (the pod may take a minute or two to start):

```
ssh balve-master "kubectl logs -f --pod-running-timeout=120s -n strapi job/restore-strapi"
```

If it fails with `is waiting to start: ContainerCreating`, wait a moment and re-run the command.

Wait for the job to complete. If the logs show errors, read the message — it usually explains what went wrong (e.g. backup not found for the given date).

---

## Step 5: Start Strapi

Start Strapi again and wait for it to become ready:

```
ssh balve-master "kubectl scale deployment strapi -n strapi --replicas=1"
ssh balve-master "kubectl rollout status deployment/strapi -n strapi --timeout=300s"
```

---

## Step 6: Re-enable ArgoCD

Turn auto-sync back on so ArgoCD manages Strapi again:

```
ssh balve-master "kubectl patch app strapi -n argocd --type merge -p '{\"spec\":{\"syncPolicy\":{\"automated\":{\"prune\":true,\"selfHeal\":true}}}}'"
```

---

## Step 7: Clean up

Delete the restore job (it's no longer needed):

```
ssh balve-master "kubectl delete job restore-strapi -n strapi"
```

---

## Step 8: Verify

Open [https://strapi.balve.garmeres.com/admin](https://strapi.balve.garmeres.com/admin) and confirm blog posts and pages are present.

---

## Admin credentials

You normally don't need these — they are only required for managing content types and users in the admin panel. To recover access:

1. Go to [strapi.balve.garmeres.com/admin](https://strapi.balve.garmeres.com/admin)
2. Click **Forgot your password?**
3. Enter `admin@garmeres.com`
4. The reset email is sent to everyone in the `admin@garmeres.com` email group
