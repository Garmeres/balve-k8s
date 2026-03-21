# Disaster Recovery Triage

Something is broken. Follow the checks below **in order** — each step builds on the one before it.

Before you start, confirm you have the [key](../01-access-management/01-overview.md) — membership in the `admin@garmeres.com` email group and access to Hetzner Cloud. You cannot proceed without both.

---

## 1. Is app data intact?

This step does not require server access — just open these two URLs in a browser:

- **Nextcloud:** [https://balve.garmeres.com](https://balve.garmeres.com) — log in and check if files, calendar, and contacts are present.
- **Strapi:** [https://strapi.balve.garmeres.com/admin](https://strapi.balve.garmeres.com/admin) — log in and check if blog posts and pages are present.

| What you see              | What to do                                     |
| ------------------------- | ---------------------------------------------- |
| Nextcloud data is missing | → [Restore Nextcloud](02-restore-nextcloud.md) |
| Strapi data is missing    | → [Restore Strapi](03-restore-strapi.md)       |
| Both look normal          | Continue to step 2                             |
| Neither site loads at all | Continue to step 2                             |

---

## 2. Is DNS pointing to the right server?

All traffic reaches the cluster through the worker node. If the worker was recreated, it may have a new IP address, and DNS still points to the old one.

1. Log in to [console.hetzner.cloud](https://console.hetzner.cloud) with your own Hetzner account
2. Open the project → _Servers_ → `worker-1`
3. Note the **Public IPv4** address

Then check what DNS currently resolves to. On any computer, run:

```
nslookup balve.garmeres.com
```

Compare the IP in the response with the worker's public IP from Hetzner.

| What you see    | What to do                                                                                                                                                              |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| IPs match       | Continue to step 3                                                                                                                                                      |
| IPs don't match | Update the DNS records at Domeneshop — see [DNS records](../02-infrastructure/07-dns-records.md). Both `balve.garmeres.com` and `*.balve.garmeres.com` need the new IP. |

---

## 3. Can you reach the cluster?

The remaining steps require running commands on the server. To get access:

1. Log in to [console.hetzner.cloud](https://console.hetzner.cloud) with your own Hetzner account (you must be a member of the project — see [grant access](../01-access-management/03-grant-access.md) if you're not)
2. Open the project → _Servers_ → `master-1`
3. Click the **>\_ Console** icon (top right) to open a web terminal
4. Log in as `root`

> **Note:** The root password was emailed to the person who created the server. If nobody has it, you can reset it from the Hetzner console: _Servers_ → `master-1` → _Rescue_ → _Reset Root Password_.

Once logged in, run:

```
kubectl get nodes
```

This checks whether the cluster (the system that runs all apps) is alive. You should see two lines — one for `master-1` and one for `worker-1`. The important part is that both show `Ready` in the STATUS column:

```
NAME       STATUS   ROLES                  AGE    VERSION
master-1   Ready    control-plane,master   ...    ...
worker-1   Ready    <none>                 ...    ...
```

| What you see                          | What to do                                                                                                       |
| ------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| Both nodes show `Ready`               | Continue to step 4                                                                                               |
| Command fails or server doesn't exist | → [Full rebuild](05-full-rebuild.md)                                                                             |
| One node shows `NotReady`             | Wait 5 minutes and try again — it may be rebooting. If it stays `NotReady`, → [Full rebuild](05-full-rebuild.md) |

### Set up SSH access

The remaining guides (restore, re-seal) use `ssh balve-master` to run commands from your local machine. Set this up now so you can copy-paste commands directly.

**Generate an SSH key** (skip if you already have one):

```
ssh-keygen -t ed25519 -f ~/.ssh/hetzner-balve
```

Press Enter twice (no passphrase).

**Add the key to the server.** In the Hetzner web terminal you opened above, run:

```
echo '<paste the contents of ~/.ssh/hetzner-balve.pub>' >> ~/.ssh/authorized_keys
```

**Configure your local SSH.** Get the server IP from the Hetzner Console (`master-1` page) and add this to `~/.ssh/config`:

```
Host balve-master
    HostName <master-1 public IP>
    User root
    IdentityFile ~/.ssh/hetzner-balve
    IdentitiesOnly yes
```

**Test it:**

```
ssh balve-master "hostname"
```

---

## 4. Are secrets working?

Secrets (passwords, API keys, etc.) are stored encrypted in git and decrypted on the cluster by a component called sealed-secrets. If this component loses its decryption key, apps will fail to start because they can't read their passwords.

Run this command on the server:

```
kubectl get pods -n sealed-secrets
```

You should see one pod with `Running` in the STATUS column:

```
NAME                              READY   STATUS    RESTARTS   AGE
sealed-secrets-...                1/1     Running   0          ...
```

If the pod is not `Running`, wait a couple of minutes — it may recover on its own. If it doesn't, skip to the "Still stuck?" section below.

Next, check if secrets are being decrypted correctly:

```
kubectl get sealedsecrets -A -o jsonpath='{range .items[*]}{.metadata.namespace}/{.metadata.name}: {.status.conditions[0].status} {.status.conditions[0].message}{"\n"}{end}'
```

This prints one line per secret. Each line shows `True` or `False` followed by a message. When everything is working, all lines show `True`:

```
argocd/argocd-dex-github: True ...
calendar-sync/calendar-sync-s3: True ...
nextcloud/nextcloud-admin: True ...
strapi/strapi-s3: True ...
...
```

If the decryption key was lost, you will see `False` with a message about decryption:

```
strapi/strapi-s3: False no key could decrypt secret
```

| What you see                                              | What to do                                |
| --------------------------------------------------------- | ----------------------------------------- |
| All lines show `True`                                     | Secrets are fine — see "Still stuck?"     |
| Any message mentions **decryption failure** or **no key** | → [Re-seal secrets](04-reseal-secrets.md) |
| `False` with a different message                          | See "Still stuck?"                        |

---

## Still stuck?

If you've gone through all four steps and haven't found the problem, open ArgoCD — the dashboard that manages all apps:

[https://argocd.balve.garmeres.com](https://argocd.balve.garmeres.com)

Log in with GitHub (you must be a member of the [Garmeres](https://github.com/Garmeres) organization). Look for any application that does not show **Synced** and **Healthy**. Click on it and read the error details — the messages are usually specific enough to point you in the right direction.
