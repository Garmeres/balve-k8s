# Nextcloud OIDC for ArgoCD

After Nextcloud is deployed and healthy, you can add it as a login provider for ArgoCD. This lets Nextcloud users log into ArgoCD with their Nextcloud account.

## Prerequisites

- ArgoCD is running ([01-deploy-argocd.md](01-deploy-argocd.md))
- Sealed secrets controller is running ([02-create-sealed-secrets.md](02-create-sealed-secrets.md))
- Nextcloud is synced and healthy in ArgoCD

---

## Register ArgoCD as an OAuth 2.0 client

Get the Nextcloud admin password. **On the server** (Hetzner console → `master-1`):

```
kubectl get secret nextcloud-admin -n nextcloud -o jsonpath='{.data.password}' | base64 -d
```

Log in to [https://balve.garmeres.com](https://balve.garmeres.com) with username `admin` and the password above.

Go to _Administration settings_ → _Security_ → _OAuth 2.0 clients_ and add a new client:

| Field        | Value                                                |
| ------------ | ---------------------------------------------------- |
| Name         | `ArgoCD`                                             |
| Redirect URI | `https://argocd.balve.garmeres.com/api/dex/callback` |

Copy the **Client ID** and **Client Secret**.

## Create the sealed secret

**On your local machine** (Linux or macOS), from the root of the `balve-k8s` repo:

```
NEXTCLOUD_CLIENT_ID='<Client ID>'
NEXTCLOUD_CLIENT_SECRET='<Client Secret>'

kubectl create secret generic argocd-dex-nextcloud --namespace argocd --dry-run=client \
  --from-literal=clientID="$NEXTCLOUD_CLIENT_ID" \
  --from-literal=clientSecret="$NEXTCLOUD_CLIENT_SECRET" \
  -o yaml | kubeseal --cert ~/sealed-secrets-cert.pem --format yaml \
  > argo-cd/templates/sealed-argocd-dex-nextcloud.yaml
```

## Commit and push

```
git add argo-cd/templates/sealed-argocd-dex-nextcloud.yaml
git commit -m "Add Nextcloud Dex sealed secret"
git push
```

## Reapply ArgoCD

ArgoCD does not reconcile itself, so reapply it to pick up the new secret. **On the server** (Hetzner console → `master-1`):

```
cd ~/balve-k8s && git fetch origin && git reset --hard origin/main
helm template argocd argo-cd -n argocd | kubectl apply -n argocd --server-side -f -
```

## Verify

Open [https://argocd.balve.garmeres.com](https://argocd.balve.garmeres.com) and log in with Nextcloud.
