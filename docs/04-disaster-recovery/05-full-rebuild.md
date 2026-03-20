# Full Rebuild

The server is gone. Start from scratch, then restore data from backups.

---

## Step 1: Rebuild infrastructure

Follow the infrastructure docs in order:

1. [Networks](../02-infrastructure/01-networks.md)
2. [Servers & placement groups](../02-infrastructure/02-servers-placement-groups.md)
3. [Firewalls](../02-infrastructure/03-firewalls.md)
4. [Server — master node](../02-infrastructure/04-server-master-node.md)
5. [Server — worker node](../02-infrastructure/05-server-worker-node.md)
6. [DNS records](../02-infrastructure/06-dns-records.md)

Skip [07-create-object-storage.md](../02-infrastructure/07-create-object-storage.md) — the S3 buckets and data still exist.

## Step 2: Deploy ArgoCD

Follow [01-deploy-argocd.md](../03-deployment/01-deploy-argocd.md).

## Step 3: Re-seal secrets

Follow [02-create-sealed-secrets.md](../03-deployment/02-create-sealed-secrets.md). The new cluster has a new signing key — all secrets must be re-sealed.

## Step 4: Restore data

1. [Restore Nextcloud](02-restore-nextcloud.md)
2. [Restore Strapi](03-restore-strapi.md)

## Step 5: Nextcloud OIDC

Follow [03-nextcloud-oidc.md](../03-deployment/03-nextcloud-oidc.md).
