# Full Rebuild

The servers are gone or unrecoverable. This guide rebuilds everything from scratch and restores data from S3 backups.

Backups are stored in Hetzner Object Storage, which is separate from the servers — even if both servers are deleted, the backups survive.

You will need:

- A Hetzner Cloud account with access to the project (see [grant access](../01-access-management/03-grant-access.md))
- A Domeneshop login (for DNS — see [access overview](../01-access-management/01-overview.md))
- A GitHub account in the [Garmeres](https://github.com/Garmeres) organization
- A local machine (Linux or macOS) with [kubectl](https://kubernetes.io/docs/tasks/tools/) and [kubeseal](https://github.com/bitnami-labs/sealed-secrets#kubeseal) installed

This takes 1–2 hours.

---

## Step 1: Rebuild infrastructure

Go through the infrastructure docs in order. Some of these resources (networks, firewalls, placement groups) may still exist — you don't need to recreate them if they're already there. Check each one in the Hetzner console and only create what's missing. If something exists but looks different from what the doc describes, update it to match.

1. [Networks](../02-infrastructure/01-networks.md)
2. [Servers & placement groups](../02-infrastructure/02-servers-placement-groups.md)
3. [Firewalls](../02-infrastructure/03-firewalls.md)
4. [SSH keys](../02-infrastructure/04-ssh-keys.md)
5. [Server — master node](../02-infrastructure/05-server-master-node.md)
6. [Server — worker node](../02-infrastructure/06-server-worker-node.md)
7. [DNS records](../02-infrastructure/07-dns-records.md) — make sure `balve.garmeres.com` and `*.balve.garmeres.com` point to the worker's public IP

Skip [Object storage](../02-infrastructure/08-create-object-storage.md) — the S3 buckets already exist and contain the backups you'll restore from.

> **After this step:** Two servers are running, but no applications are deployed yet.

---

## Step 2: Deploy ArgoCD

ArgoCD is the system that deploys and manages all applications. It needs to be installed manually first — everything else it handles automatically.

Follow [Deploy ArgoCD](../03-deployment/01-deploy-argocd.md). This runs on the server via the Hetzner web console.

> **After this step:** ArgoCD is running and will try to deploy all applications, but they will fail because the secrets don't exist yet.

---

## Step 3: Re-seal secrets

The new cluster has a new encryption key, so all secrets need to be re-encrypted.

Follow [Sealed Secrets](../03-deployment/02-create-sealed-secrets.md). This involves fetching credentials from Hetzner, Domeneshop, and GitHub, then running commands on your local machine.

> **After this step:** All applications should start deploying successfully. Check progress at [argocd.balve.garmeres.com](https://argocd.balve.garmeres.com). If any app is stuck syncing or shows "retry limit exceeded," click **Terminate** on the sync in the ArgoCD dashboard, then click **Sync** to start a fresh sync with the updated secrets. Pods may take a few minutes to become healthy.

---

## Step 4: Restore data

The applications are running but have empty databases. Restore from the S3 backups:

1. [Restore Nextcloud](02-restore-nextcloud.md) — restores the MariaDB database, configuration, and user files
2. [Restore Strapi](03-restore-strapi.md) — restores the SQLite database

> **After this step:** Open [balve.garmeres.com](https://balve.garmeres.com) and [strapi.balve.garmeres.com/admin](https://strapi.balve.garmeres.com/admin) to confirm everything is back.

---

## Step 5: Nextcloud OIDC

This lets users log into ArgoCD with their Nextcloud account. It's optional — GitHub login already works after step 3 — but it should be set up for completeness.

Follow [Nextcloud OIDC for ArgoCD](../03-deployment/03-nextcloud-oidc.md). This requires registering ArgoCD as an OAuth client in Nextcloud's admin panel.
