# Re-seal Secrets

The sealed-secrets controller lost its signing key (e.g. PVC deleted, controller redeployed). All SealedSecret resources become undecryptable — pods fail with missing secret errors.

All credentials are recoverable. This takes ~30 minutes.

---

## Confirm the problem

On the server (go to [console.hetzner.cloud](https://console.hetzner.cloud) → _Servers_ → `master-1` → **>_ Console** → log in as `root`):

```
kubectl get sealedsecrets -A -o jsonpath='{range .items[*]}{.metadata.namespace}/{.metadata.name}: {.status.conditions[0].status} {.status.conditions[0].message}{"\n"}{end}'
```

If any show `False` with a message about decryption failure, the signing key was lost.

## Re-seal

Follow [02-create-sealed-secrets.md](../03-deployment/02-create-sealed-secrets.md) from the beginning. The new controller has a new signing key, so all secrets must be re-sealed with the new cert.

## Verify

Once the push lands and ArgoCD syncs, all pods should recover:

```
kubectl get pods -A
```

If Nextcloud or Strapi data is also missing, continue with [Restore Nextcloud](02-restore-nextcloud.md) and/or [Restore Strapi](03-restore-strapi.md).
