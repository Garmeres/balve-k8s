# Sealed Secrets

Secrets are encrypted with [Sealed Secrets](https://sealed-secrets.netlify.app/) and stored in git. The controller in the cluster decrypts them.

## Install tools

```
brew install kubectl kubeseal
```

## Wait for the controller

After deploying ArgoCD (doc 10), the sealed-secrets controller is deployed automatically. Wait for it:

```
ssh balve-master 'kubectl get pods -n sealed-secrets -w'
```

---

## Path A: Restore existing signing key

Use this path if you have a `signing-key-backup.yaml` from a previous cluster. The sealed secrets already committed in git will be decryptable.

Restore the key **before** the controller starts (or restart it after):

```
cat signing-key-backup.yaml | ssh balve-master 'kubectl apply -f -'
ssh balve-master 'kubectl rollout restart deployment -n sealed-secrets sealed-secrets'
```

The controller picks up the restored key and decrypts the SealedSecrets already in git. **You're done.**

---

## Path B: Create new secrets from scratch

Use this path on first setup, or if the signing key is lost. All secrets will be regenerated.

### Backup the signing key

The controller's signing key is the only thing that can decrypt your secrets. Save it somewhere safe (password manager):

```
ssh balve-master 'kubectl get secret -n sealed-secrets -l sealedsecrets.bitnami.com/sealed-secrets-key -o yaml' > signing-key-backup.yaml
```

**Do not commit this file to git.** Store it in a password manager or other secure location.

### Credentials

In the Hetzner console, go to _Object Storage_ -> _Manage credentials_ -> _Generate credentials_.

Go to the Domeneshop control panel -> _E-post_ -> _SMTP-innstillinger_ and note the username and password.

```
S3_KEY="<S3 Access Key>"
S3_SECRET="<S3 Secret Key>"
SMTP_USER="<SMTP username>"
SMTP_PASS="<SMTP password>"
```

### Fetch the public cert

```
ssh balve-master 'kubectl get secret -n sealed-secrets -l sealedsecrets.bitnami.com/sealed-secrets-key -o jsonpath="{.items[0].data.tls\.crt}"' \
  | base64 -d > sealed-secrets-cert.pem
```

### Seal function

Add this helper to your terminal session:

```
seal() {
  local app="$1" name="$2"
  shift 2
  kubectl create secret generic "$name" --namespace "$app" --dry-run=client "$@" -o yaml \
    | kubeseal --cert sealed-secrets-cert.pem --format yaml \
    > applications/"$app"/templates/sealed-"$name".yaml
}
```

Run all commands below from the **root of the balve-k8s repo**.

### Strapi

```
seal strapi strapi-s3 \
  --from-literal=S3_ACCESS_KEY_ID="$S3_KEY" \
  --from-literal=S3_SECRET_ACCESS_KEY="$S3_SECRET"

seal strapi strapi-secrets \
  --from-literal=APP_KEYS="$(openssl rand -base64 16),$(openssl rand -base64 16)" \
  --from-literal=API_TOKEN_SALT="$(openssl rand -base64 16)" \
  --from-literal=ADMIN_JWT_SECRET="$(openssl rand -base64 16)" \
  --from-literal=TRANSFER_TOKEN_SALT="$(openssl rand -base64 16)" \
  --from-literal=ENCRYPTION_KEY="$(openssl rand -base64 16)"
```

### Nextcloud

```
seal nextcloud nextcloud-s3 \
  --from-literal=S3_ACCESS_KEY_ID="$S3_KEY" \
  --from-literal=S3_SECRET_ACCESS_KEY="$S3_SECRET"

seal nextcloud nextcloud-admin \
  --from-literal=username="admin" \
  --from-literal=password="$(openssl rand -base64 16)" \
  --from-literal=smtp-username="$SMTP_USER" \
  --from-literal=smtp-password="$SMTP_PASS"

seal nextcloud nextcloud-mariadb \
  --from-literal=mariadb-root-password="$(openssl rand -base64 16)" \
  --from-literal=mariadb-password="$(openssl rand -base64 16)"
```

### Calendar Sync

```
seal calendar-sync calendar-sync-s3 \
  --from-literal=S3_ACCESS_KEY_ID="$S3_KEY" \
  --from-literal=S3_SECRET_ACCESS_KEY="$S3_SECRET"
```

### Commit and push

```
git add applications/*/templates/sealed-*.yaml
git commit -m "Add sealed secrets"
git push
```

ArgoCD will sync the SealedSecret resources. The controller decrypts them into regular Kubernetes Secrets in each namespace.
