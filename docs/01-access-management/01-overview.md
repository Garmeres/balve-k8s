# Access management

The board assigns one member as **teknisk ansvarlig**. This person holds root and infrastructure access and an admin account in each application. At least one other board member should also have root access, so it is not lost in an emergency.

Access to the Balve platform is organized in five levels. Each level is granted by the one above it.

| Level                     | What                              |
| ------------------------- | --------------------------------- |
| 1. Root                   | `admin@garmeres.com` email group  |
| 2. Infrastructure         | Domeneshop, Hetzner Cloud, GitHub |
| 3. Application superadmin | Nextcloud, Strapi                 |
| 4. Admin user             | App users with admin privileges   |
| 5. User                   | Individual accounts               |

## 1. Root

Membership in the email group `admin@garmeres.com`. Emails sent here are forwarded to each member's personal email address.

This is the root of trust for the entire platform. Everything else — Domeneshop, Hetzner, Strapi superadmin — is recoverable from it. Root access is per-person and cannot be recovered on its own. To get it, someone who already has it must invite you.

## 2. Infrastructure

### Domeneshop

- **What it controls:** DNS records, email (SMTP), and the `admin@garmeres.com` mailbox for `garmeres.com`
- **Login:** [domene.shop/login](https://domene.shop/login)
- **Username:** `admin@garmeres.com`
- **Password:** Stored in Balve passwords. If Balve passwords is down, anyone in the `admin@garmeres.com` email group can reset the password.

### Hetzner Cloud

- **What it controls:** Servers, firewalls, networks, object storage (S3)
- **Login:** [console.hetzner.cloud](https://console.hetzner.cloud)
- **Project:** 12640153
- Each person has their own Hetzner account, invited to the project.

### GitHub

Write access to the [Garmeres GitHub organization](https://github.com/Garmeres). Controls all cluster configuration, Helm charts, sealed secrets, and documentation. ArgoCD login is automatic for organization members. In an emergency, the repositories can be forked — GitHub access is not required for recovery.

## 3. Application superadmin

Emergency accounts. Only needed if all admin users are gone or have lost their access.

### Nextcloud

- **Admin panel:** [balve.garmeres.com](https://balve.garmeres.com)
- **Username:** `admin`
- **Password:** Stored in a Kubernetes secret. Requires server access (Hetzner) to retrieve.

### Strapi

- **Admin panel:** [strapi.balve.garmeres.com/admin](https://strapi.balve.garmeres.com/admin)
- **Email:** `admin@garmeres.com`
- **Password reset:** Goes to the `admin@garmeres.com` email group.

## 4. Admin user

Users with admin privileges within each application. Can manage other users and settings. Can be granted by another admin user or by the application superadmin.

- **Nextcloud:** Add a user to the admin group in Settings → Users.
- **Strapi:** Assign the admin role in Settings → Administration panel.

## 5. User

Individual accounts for using the services. Created by an admin user.

- **Email:** Personal `@garmeres.com` address (e.g. `fornavn@garmeres.com`) — forwards to the person's personal email
- **Nextcloud:** Personal account at [balve.garmeres.com](https://balve.garmeres.com) — files, calendar, contacts
- **Strapi:** Editor account at [strapi.balve.garmeres.com/admin](https://strapi.balve.garmeres.com/admin) — content management

## Guides

1. [Handover](02-handover.md) — Transfer the teknisk ansvarlig role to a new person
2. [Grant access](03-grant-access.md) — Detailed reference for granting access at each level
3. [Revoke access](04-revoke-access.md) — Detailed reference for revoking access at each level
4. [Audit access](05-audit-access.md) — Check who currently has access
