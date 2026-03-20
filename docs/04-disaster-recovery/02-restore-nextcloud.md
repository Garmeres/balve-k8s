# Restore Nextcloud

Restores the Nextcloud MariaDB database and `config.php` from an S3 backup. Works whether the data is corrupt or the PVC was deleted.

All commands run on the server. To access it:

1. Go to [console.hetzner.cloud](https://console.hetzner.cloud) and log in
2. Open the project → _Servers_ → `master-1`
3. Click the **>\_ Console** icon (top right) to open a web terminal
4. Log in as `root`

---

## Ensure resources exist

Trigger an ArgoCD sync (no-op if nothing was deleted):

```
kubectl patch app nextcloud -n argocd --type merge \
  -p '{"operation":{"initiatedBy":{"username":"admin"},"sync":{}}}'
```

Wait for PVCs and MariaDB to be ready:

```
kubectl wait --for=jsonpath=.status.phase=Bound pvc/nextcloud-nextcloud \
  pvc/data-nextcloud-mariadb-0 -n nextcloud --timeout=60s
kubectl rollout status statefulset/nextcloud-mariadb -n nextcloud --timeout=120s
```

## Pause ArgoCD

Pause auto-sync so scale-down is not reverted:

```
kubectl patch app nextcloud -n argocd --type merge \
  -p '{"spec":{"syncPolicy":null}}'
```

## Scale down Nextcloud

MariaDB stays running for the database import:

```
kubectl scale deployment nextcloud -n nextcloud --replicas=0
```

## Restore

Restore the latest backup:

```
kubectl create job --from=cronjob/nextcloud-restore restore-nextcloud -n nextcloud
```

Or restore a specific date (YYYYMMDD). Available dates are listed in the job logs, or in the Hetzner Object Storage console under `balve-nextcloud/backups/`:

```
kubectl create job --from=cronjob/nextcloud-restore restore-nextcloud -n nextcloud \
  --dry-run=client -o json | \
  jq '(.spec.template.spec.containers[0].env[] | select(.name == "RESTORE_DATE")).value = "20260317"' | \
  kubectl apply -f -
```

Follow the job logs:

```
kubectl logs -f --pod-running-timeout=60s -n nextcloud job/restore-nextcloud
```

## Scale up

```
kubectl scale deployment nextcloud -n nextcloud --replicas=1
kubectl rollout restart deployment/nextcloud -n nextcloud
kubectl rollout status deployment/nextcloud -n nextcloud --timeout=300s
```

## Re-enable ArgoCD

```
kubectl patch app nextcloud -n argocd --type merge \
  -p '{"spec":{"syncPolicy":{"automated":{"prune":true,"selfHeal":true}}}}'
```

## Admin credentials

The Nextcloud admin credentials are normally not needed — users log in with their own accounts. If you need them (e.g. to manage users or change settings):

- **Username:** `admin`
- **Password:** Read it from the Kubernetes secret:

```
kubectl get secret nextcloud-admin -n nextcloud -o jsonpath='{.data.password}' | base64 -d
```

If the password doesn't work (e.g. it was changed manually), reset it:

```
kubectl exec -it deploy/nextcloud -n nextcloud -- su -s /bin/bash www-data -c "php occ user:resetpassword admin"
```

## Clean up

```
kubectl delete job restore-nextcloud -n nextcloud
```

## Verify

Open [https://balve.garmeres.com](https://balve.garmeres.com) and confirm files, calendar, and contacts are present.
