# Re-seal Secrets

The sealed-secrets controller lost its signing key. This means all encrypted secrets in git can no longer be decrypted — apps fail to start because they can't read their passwords and API keys.

All credentials are recoverable. This procedure takes about 30 minutes and requires:

- Server access (Hetzner Cloud)
- A local machine with [kubectl](https://kubernetes.io/docs/tasks/tools/) and [kubeseal](https://github.com/bitnami-labs/sealed-secrets#kubeseal) installed
- Access to Hetzner Cloud Console, Domeneshop, GitHub, and (if still in use) AWS

---

## 1. Confirm the problem

On the server (go to [console.hetzner.cloud](https://console.hetzner.cloud) → _Servers_ → `master-1` → **>\_ Console** → log in as `root`):

```
kubectl get sealedsecrets -A -o jsonpath='{range .items[*]}{.metadata.namespace}/{.metadata.name}: {.status.conditions[0].status} {.status.conditions[0].message}{"\n"}{end}'
```

If any line shows `False` with a message about decryption failure or no matching key, the signing key was lost. For example:

```
strapi/strapi-s3: False no key could decrypt secret
```

If the messages mention something else, this is not a signing key problem — go back to [triage](01-triage.md).

---

## 2. Re-seal all secrets

The sealed-secrets controller generated a new signing key when it was redeployed. You need to re-encrypt every secret with the new key.

Follow the full [Sealed Secrets deployment guide](../03-deployment/02-create-sealed-secrets.md) from the beginning. It walks you through each secret step by step:

1. **Export the new public cert** from the cluster (uploaded to `balve-config` bucket, downloaded from Hetzner console)
2. **Regenerate S3 credentials** at Hetzner (Object Storage → Manage credentials)
3. **Reset the SMTP password** at Domeneshop (garmeres10 email account)
4. **Regenerate the GitHub OAuth secret** for ArgoCD
5. **Regenerate random secrets** for Strapi (app keys, JWT) and Nextcloud (MariaDB, Redis passwords)
6. **Regenerate AWS credentials** for calendar-sync (if still in use)
7. **Commit and push** — ArgoCD syncs the new sealed secrets automatically

> **Note:** User accounts, files, and data are not affected by this process. No one's login will change. Some services will restart automatically — this is normal.

---

## 3. Verify

After pushing, wait for ArgoCD to sync. You can watch the progress at [argocd.balve.garmeres.com](https://argocd.balve.garmeres.com) or on the server:

```
kubectl get pods -A -w
```

All pods should eventually show `Running`. This may take a few minutes as apps restart with the new secrets.

If pods are still failing after 5 minutes, check the ArgoCD dashboard for specific error messages.

---

## 4. Check for data loss

Re-sealing secrets does not cause data loss on its own, but the original incident (e.g. the controller being deleted or the cluster being rebuilt) might have. If data is also missing:

- [Restore Nextcloud](02-restore-nextcloud.md)
- [Restore Strapi](03-restore-strapi.md)
