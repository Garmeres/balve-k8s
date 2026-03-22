# balve-k8s

Kubernetes infrastructure for [Garmeres](https://garmeres.com). Runs on two Hetzner Cloud servers with [k3s](https://k3s.io), managed by [ArgoCD](https://argo-cd.readthedocs.io/).

## Applications

| App                                           | Description                                                          |
| --------------------------------------------- | -------------------------------------------------------------------- |
| [strapi](applications/strapi)                 | Headless CMS for garmeres.com (Strapi 5, SQLite)                     |
| [nextcloud](applications/nextcloud)           | File sharing, calendar, collaboration (MariaDB, Collabora)           |
| [calendar-sync](applications/calendar-sync)   | CronJob that syncs calendar as JSON to Hetzner S3 and AWS CloudFront |
| [cert-manager](applications/cert-manager)     | TLS certificates via Let's Encrypt                                   |
| [sealed-secrets](applications/sealed-secrets) | Encrypted secrets stored in git                                      |
| [argocd-config](applications/argocd-config)   | Sealed secrets for ArgoCD Dex OAuth                                  |

## Need to grant or revoke access?

If you're a technical manager and need to give someone access to Balve (e.g. a consultant or new board member), or remove access from someone, see the [access management guide](docs/01-access-management/01-access-management.md).

## Architecture

- **Cluster:** k3s on Hetzner Cloud (1 master + 1 worker)
- **Ingress:** Traefik (built into k3s)
- **TLS:** cert-manager with Let's Encrypt HTTP-01
- **GitOps:** ArgoCD ApplicationSet discovers all apps under `applications/`
- **Secrets:** Sealed Secrets — encrypted in git, decrypted by controller
- **Storage:** Hetzner Object Storage (S3-compatible) for media and backups

## Documentation

### [1. Access management](docs/01-access-management/)

1. [Access management](docs/01-access-management/01-access-management.md)
2. [Rotate SSH keys](docs/01-access-management/02-rotate-ssh-keys.md)
3. [New board member](docs/01-access-management/03-new-board-member.md)
4. [Outgoing board member](docs/01-access-management/04-outgoing-board-member.md)
5. [New external collaborator](docs/01-access-management/05-new-external-collaborator.md)
6. [Outgoing external collaborator](docs/01-access-management/06-outgoing-external-collaborator.md)
7. [New technical manager](docs/01-access-management/07-new-technical-manager.md)
8. [Outgoing technical manager](docs/01-access-management/08-outgoing-technical-manager.md)
9. [Manage Balve users](docs/01-access-management/09-manage-balve-users.md)
10. [Manage email users](docs/01-access-management/10-manage-email-users.md)
11. [Manage Strapi users](docs/01-access-management/11-manage-strapi-users.md)
12. [Manage GitHub users](docs/01-access-management/12-manage-github-users.md)
13. [Manage Hetzner users](docs/01-access-management/13-manage-hetzner-users.md)
14. [Access structure](docs/01-access-management/14-access-structure.md)
15. [Audit access](docs/01-access-management/15-audit-access.md)

### [2. Balve infrastructure](docs/02-balve-infrastructure/)

1. [Networks](docs/02-balve-infrastructure/01-networks.md)
2. [Placement groups](docs/02-balve-infrastructure/02-servers-placement-groups.md)
3. [Firewalls](docs/02-balve-infrastructure/03-firewalls.md)
4. [SSH keys](docs/02-balve-infrastructure/04-ssh-keys.md)
5. [Master node](docs/02-balve-infrastructure/05-server-master-node.md)
6. [Worker node](docs/02-balve-infrastructure/06-server-worker-node.md)
7. [DNS records](docs/02-balve-infrastructure/07-dns-records.md)
8. [Object storage](docs/02-balve-infrastructure/08-create-object-storage.md)

### [3. Balve deployment](docs/03-balve-deployment/)

1. [Deploy ArgoCD](docs/03-balve-deployment/01-deploy-argocd.md)
2. [Sealed secrets](docs/03-balve-deployment/02-create-sealed-secrets.md)
3. [Nextcloud OIDC for ArgoCD](docs/03-balve-deployment/03-nextcloud-oidc.md)

### [4. Disaster recovery](docs/04-disaster-recovery/)

1. [Triage](docs/04-disaster-recovery/01-triage.md)
2. [Restore Nextcloud](docs/04-disaster-recovery/02-restore-nextcloud.md)
3. [Restore Strapi](docs/04-disaster-recovery/03-restore-strapi.md)
4. [Reseal secrets](docs/04-disaster-recovery/04-reseal-secrets.md)
5. [Full rebuild](docs/04-disaster-recovery/05-full-rebuild.md)
