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

## Architecture

- **Cluster:** k3s on Hetzner Cloud (1 master + 1 worker)
- **Ingress:** Traefik (built into k3s)
- **TLS:** cert-manager with Let's Encrypt HTTP-01
- **GitOps:** ArgoCD ApplicationSet discovers all apps under `applications/`
- **Secrets:** Sealed Secrets — encrypted in git, decrypted by controller
- **Storage:** Hetzner Object Storage (S3-compatible) for media, backups, and Litestream replication

## Setup guide

### Day 0 — Infrastructure

1. [Networks](docs/day-0/01-networks.md)
2. [Placement groups](docs/day-0/02-servers-placement-groups.md)
3. [Firewalls](docs/day-0/03-firewalls.md)
4. [SSH key](docs/day-0/04-create-ssh-key.md)
5. [Master node](docs/day-0/05-server-master-node.md)
6. [SSH config](docs/day-0/06-ssh-config-for-master-1.md)
7. [Worker node](docs/day-0/07-server-worker-node.md)
8. [DNS records](docs/day-0/08-dns-records.md)
9. [Object storage](docs/day-0/09-create-object-storage.md)

### Day 1 — Platform

1. [Deploy ArgoCD](docs/day-1/01-deploy-argocd.md)
2. [Sealed secrets](docs/day-1/02-create-sealed-secrets.md)
