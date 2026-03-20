# Sealed Secrets

Secrets are encrypted with [Sealed Secrets](https://sealed-secrets.netlify.app/) and stored in git. The controller in the cluster decrypts them.

## Export the public cert

**On the server** (Hetzner console → `master-1`):

Wait for ArgoCD to sync the sealed-secrets application:

```
kubectl get application sealed-secrets -n argocd -w
```

Once it shows `Synced`, wait for the controller pod:

```
kubectl get pods -n sealed-secrets -w
```

Once it is up, print the public certificate:

```
kubeseal --fetch-cert --controller-namespace sealed-secrets
```

Copy the output (starts with `-----BEGIN CERTIFICATE-----`) and save it on your local machine as `~/sealed-secrets-cert.pem`.

---

## Seal the secrets

All remaining commands run **on your local machine** (Linux or macOS), from the root of the `balve-k8s` repo.

### Install tools

Install [kubectl](https://kubernetes.io/docs/tasks/tools/) and [kubeseal](https://github.com/bitnami-labs/sealed-secrets#kubeseal) on your local machine.

### S3

In the [Hetzner Cloud Console](https://console.hetzner.cloud), go to _Object Storage_ → _Manage credentials_ → _Generate credentials_.

```
S3_KEY='<S3 Access Key>'
S3_SECRET='<S3 Secret Key>'
```

```
kubectl create secret generic strapi-s3 --namespace strapi --dry-run=client \
  --from-literal=S3_ACCESS_KEY_ID="$S3_KEY" \
  --from-literal=S3_SECRET_ACCESS_KEY="$S3_SECRET" \
  -o yaml | kubeseal --cert ~/sealed-secrets-cert.pem --format yaml \
  > applications/strapi/templates/sealed-strapi-s3.yaml

kubectl create secret generic nextcloud-s3 --namespace nextcloud --dry-run=client \
  --from-literal=S3_ACCESS_KEY_ID="$S3_KEY" \
  --from-literal=S3_SECRET_ACCESS_KEY="$S3_SECRET" \
  -o yaml | kubeseal --cert ~/sealed-secrets-cert.pem --format yaml \
  > applications/nextcloud/templates/sealed-nextcloud-s3.yaml

kubectl create secret generic calendar-sync-s3 --namespace calendar-sync --dry-run=client \
  --from-literal=S3_ACCESS_KEY_ID="$S3_KEY" \
  --from-literal=S3_SECRET_ACCESS_KEY="$S3_SECRET" \
  -o yaml | kubeseal --cert ~/sealed-secrets-cert.pem --format yaml \
  > applications/calendar-sync/templates/sealed-calendar-sync-s3.yaml
```

### SMTP

In the [Domeneshop control panel](https://domene.shop/login), log in with **admin@garmeres.com**. Go to _Mine domener_ → _garmeres.com_ → _Epost_, click **Balve** (`garmeres10`), and reset the password.

```
SMTP_PASS='<new password>'
```

```
kubectl create secret generic strapi-smtp --namespace strapi --dry-run=client \
  --from-literal=SMTP_USERNAME="garmeres10" \
  --from-literal=SMTP_PASSWORD="$SMTP_PASS" \
  -o yaml | kubeseal --cert ~/sealed-secrets-cert.pem --format yaml \
  > applications/strapi/templates/sealed-strapi-smtp.yaml

kubectl create secret generic nextcloud-admin --namespace nextcloud --dry-run=client \
  --from-literal=username="admin" \
  --from-literal=password="$(openssl rand -base64 16)" \
  --from-literal=smtp-host="smtp.domeneshop.no" \
  --from-literal=smtp-username="garmeres10" \
  --from-literal=smtp-password="$SMTP_PASS" \
  -o yaml | kubeseal --cert ~/sealed-secrets-cert.pem --format yaml \
  > applications/nextcloud/templates/sealed-nextcloud-admin.yaml
```

### GitHub OAuth

Go to [github.com/organizations/garmeres/settings/applications](https://github.com/organizations/garmeres/settings/applications). If the **ArgoCD** OAuth App already exists, open it and generate a new client secret. Otherwise, create it:

| Field                      | Value                                                |
| -------------------------- | ---------------------------------------------------- |
| Application name           | `ArgoCD`                                             |
| Homepage URL               | `https://argocd.balve.garmeres.com`                  |
| Authorization callback URL | `https://argocd.balve.garmeres.com/api/dex/callback` |
| Enable Device Flow         | checked                                              |

Click _Register application_.

In both cases, the **Client ID** is shown near the top of the app page. Click _Generate a new client secret_ — the secret is only shown once, copy it immediately.

```
GITHUB_CLIENT_ID='<Client ID>'
GITHUB_CLIENT_SECRET='<Client Secret>'
```

```
kubectl create secret generic argocd-dex-github --namespace argocd --dry-run=client \
  --from-literal=clientID="$GITHUB_CLIENT_ID" \
  --from-literal=clientSecret="$GITHUB_CLIENT_SECRET" \
  -o yaml | kubeseal --cert ~/sealed-secrets-cert.pem --format yaml \
  > applications/argocd-config/templates/sealed-argocd-dex-github.yaml
```

### Generated secrets

These use randomly generated values — no credentials to gather.

```
kubectl create secret generic strapi-secrets --namespace strapi --dry-run=client \
  --from-literal=APP_KEYS="$(openssl rand -base64 16),$(openssl rand -base64 16)" \
  --from-literal=API_TOKEN_SALT="$(openssl rand -base64 16)" \
  --from-literal=ADMIN_JWT_SECRET="$(openssl rand -base64 16)" \
  --from-literal=TRANSFER_TOKEN_SALT="$(openssl rand -base64 16)" \
  --from-literal=ENCRYPTION_KEY="$(openssl rand -base64 16)" \
  -o yaml | kubeseal --cert ~/sealed-secrets-cert.pem --format yaml \
  > applications/strapi/templates/sealed-strapi-secrets.yaml

kubectl create secret generic nextcloud-mariadb --namespace nextcloud --dry-run=client \
  --from-literal=mariadb-root-password="$(openssl rand -base64 16)" \
  --from-literal=mariadb-password="$(openssl rand -base64 16)" \
  --from-literal=db-username="nextcloud" \
  -o yaml | kubeseal --cert ~/sealed-secrets-cert.pem --format yaml \
  > applications/nextcloud/templates/sealed-nextcloud-mariadb.yaml

kubectl create secret generic nextcloud-redis --namespace nextcloud --dry-run=client \
  --from-literal=redis-password="$(openssl rand -base64 24)" \
  -o yaml | kubeseal --cert ~/sealed-secrets-cert.pem --format yaml \
  > applications/nextcloud/templates/sealed-nextcloud-redis.yaml
```

### AWS S3 (backwards compat)

Temporary secret for syncing to the old AWS S3 bucket + CloudFront. Remove this when the old frontend is retired.

Create an IAM user `calendar-sync-k8s` in the AWS Console with a policy allowing `s3:PutObject`, `s3:GetObject`, `s3:DeleteObject`, `s3:ListBucket` on `garmeres-calendar-sync-events-bucket` and `cloudfront:CreateInvalidation`. Generate an access key.

```
AWS_KEY='<AWS Access Key ID>'
AWS_SECRET='<AWS Secret Access Key>'
```

```
kubectl create secret generic calendar-sync-aws-s3 --namespace calendar-sync --dry-run=client \
  --from-literal=S3_ACCESS_KEY_ID="$AWS_KEY" \
  --from-literal=S3_SECRET_ACCESS_KEY="$AWS_SECRET" \
  -o yaml | kubeseal --cert ~/sealed-secrets-cert.pem --format yaml \
  > applications/calendar-sync/templates/sealed-calendar-sync-aws-s3.yaml
```

### Commit and push

```
git add applications/*/templates/sealed-*.yaml
git commit -m "Add sealed secrets"
git push
```

ArgoCD will sync the SealedSecret resources. The controller decrypts them into regular Kubernetes Secrets in each namespace. Applications that depend on these secrets (Strapi, Nextcloud, calendar-sync) will start recovering automatically.
