# Nextcloud OIDC for ArgoCD

After Nextcloud is deployed and healthy, you can add it as a login provider for ArgoCD. This lets Nextcloud users log into ArgoCD with their Nextcloud account.

## Prerequisites

- ArgoCD is running ([01-deploy-argocd.md](01-deploy-argocd.md))
- Sealed secrets controller is running ([02-create-sealed-secrets.md](02-create-sealed-secrets.md))
- Nextcloud is synced and healthy in ArgoCD

---

## Register ArgoCD as an OIDC client

Delete any existing ArgoCD client, then create a new one via the OIDC app admin API:

```
ssh balve-master "ADMIN_PASS=\$(kubectl get secret nextcloud-admin -n nextcloud -o jsonpath='{.data.password}' | base64 -d) && \
  DB_PASS=\$(kubectl get secret nextcloud-mariadb -n nextcloud -o jsonpath='{.data.mariadb-password}' | base64 -d) && \
  EXISTING=\$(kubectl exec statefulset/nextcloud-mariadb -n nextcloud -- mariadb -u nextcloud -p\$DB_PASS nextcloud -N -e \"SELECT id FROM oc_oidc_clients WHERE name='ArgoCD';\") && \
  for id in \$EXISTING; do kubectl exec deploy/nextcloud -n nextcloud -- curl -s -u admin:\$ADMIN_PASS -H 'OCS-APIREQUEST: true' -X DELETE \"http://localhost/index.php/apps/oidc/clients/\$id\"; done && \
  kubectl exec deploy/nextcloud -n nextcloud -- curl -s -u admin:\$ADMIN_PASS -H 'OCS-APIREQUEST: true' -X POST 'http://localhost/index.php/apps/oidc/clients' -d 'name=ArgoCD&redirectUri=https://argocd.balve.garmeres.com/api/dex/callback&signingAlg=RS256&type=confidential&flowType=code'"
```

The response JSON contains `client_identifier` and `secret`. Copy both values.

## Create the sealed secret

**On your local machine** (Linux or macOS), from the root of the `balve-k8s` repo:

```
NEXTCLOUD_CLIENT_ID='<Client ID>'
NEXTCLOUD_CLIENT_SECRET='<Client Secret>'

kubectl create secret generic argocd-dex-nextcloud --namespace argocd --dry-run=client \
  --from-literal=clientID="$NEXTCLOUD_CLIENT_ID" \
  --from-literal=clientSecret="$NEXTCLOUD_CLIENT_SECRET" \
  -o yaml | \
  kubectl label --local -f - app.kubernetes.io/part-of=argocd --dry-run=client -o yaml | \
  kubeseal --cert ~/sealed-secrets-cert.pem --format yaml \
  > applications/argocd-config/templates/sealed-argocd-dex-nextcloud.yaml
```

Create a copy in the Nextcloud namespace (used by the restore job to re-register the OAuth client automatically):

```
kubectl create secret generic argocd-oauth --namespace nextcloud --dry-run=client \
  --from-literal=clientID="$NEXTCLOUD_CLIENT_ID" \
  --from-literal=clientSecret="$NEXTCLOUD_CLIENT_SECRET" \
  -o yaml | \
  kubeseal --cert ~/sealed-secrets-cert.pem --format yaml \
  > applications/nextcloud/templates/sealed-argocd-oauth.yaml
```

## Commit and push

```
git add applications/argocd-config/templates/sealed-argocd-dex-nextcloud.yaml applications/nextcloud/templates/sealed-argocd-oauth.yaml
git commit -m "Add Nextcloud Dex sealed secret"
git push
```

## Sync and restart Dex

Force a sync of `argocd-config` and `argocd`:

```
ssh balve-master "kubectl patch application argocd-config -n argocd --type merge -p '{\"operation\":{\"sync\":{\"syncStrategy\":{\"apply\":{\"force\":true}}}}}'"
ssh balve-master "kubectl patch application argocd -n argocd --type merge -p '{\"operation\":{\"sync\":{\"syncStrategy\":{\"apply\":{\"force\":true}}}}}'"
```

Restart Dex so it loads the GitHub and Nextcloud OAuth credentials:

```
ssh balve-master "kubectl rollout restart deployment argocd-dex-server -n argocd"
```

## Verify

Watch until `argocd` shows **Healthy**:

```
ssh balve-master "kubectl get applications -n argocd -w"
```

Open [https://argocd.balve.garmeres.com](https://argocd.balve.garmeres.com) and log in with Github.
