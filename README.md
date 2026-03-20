# balve-k8s

Kubernetes infrastructure for [Garmeres](https://garmeres.com). Runs on two Hetzner Cloud servers with [k3s](https://k3s.io), managed by [ArgoCD](https://argo-cd.readthedocs.io/).

## Applications

| App                                           | Description                                                |
| --------------------------------------------- | ---------------------------------------------------------- |
| [strapi](applications/strapi)                 | Headless CMS (Strapi 5, SQLite + Litestream)               |
| [nextcloud](applications/nextcloud)           | File sharing, calendar, collaboration (MariaDB, Collabora) |
| [calendar-sync](applications/calendar-sync)   | CronJob that exports calendar to S3 as JSON                |
| [cert-manager](applications/cert-manager)     | TLS certificates via Let's Encrypt                         |
| [sealed-secrets](applications/sealed-secrets) | Encrypted secrets stored in git                            |

## Need to grant or revoke access?

If you're a technical manager and need to give someone access to Balve (e.g. a consultant or new team member), or remove access from someone, see the [access management guide](docs/01-access-management/).

## Architecture

- **Cluster:** k3s on Hetzner Cloud (1 master + 1 worker)
- **Ingress:** Traefik (built into k3s)
- **TLS:** cert-manager with Let's Encrypt HTTP-01
- **GitOps:** ArgoCD ApplicationSet discovers all apps under `applications/`
- **Secrets:** Sealed Secrets — encrypted in git, decrypted by controller
- **Storage:** Hetzner Object Storage (S3-compatible) for media, backups, and Litestream replication

## Documentation

### [1. Access management](docs/01-access-management/)

Grant, revoke, and audit administrative access to the platform.

### [2. Infrastructure](docs/02-infrastructure/)

1. [Networks](docs/02-infrastructure/01-networks.md)
2. [Placement groups](docs/02-infrastructure/02-servers-placement-groups.md)
3. [Firewalls](docs/02-infrastructure/03-firewalls.md)
4. [Master node](docs/02-infrastructure/04-server-master-node.md)
5. [Worker node](docs/02-infrastructure/05-server-worker-node.md)
6. [DNS records](docs/02-infrastructure/06-dns-records.md)
7. [Object storage](docs/02-infrastructure/07-create-object-storage.md)

### [3. Deployment](docs/03-deployment/)

1. [Deploy ArgoCD](docs/03-deployment/01-deploy-argocd.md)
2. [Sealed secrets](docs/03-deployment/02-create-sealed-secrets.md)
3. [Nextcloud OIDC for ArgoCD](docs/03-deployment/03-nextcloud-oidc.md)
