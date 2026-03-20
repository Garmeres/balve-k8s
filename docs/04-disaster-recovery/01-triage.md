# Disaster Recovery Triage

Something is broken. Follow the checks below in order to identify the problem.

Before you start, confirm you have the [key](../01-access-management/01-overview.md) — membership in the `admin@garmeres.com` email group and access to Hetzner Cloud. You cannot proceed without both.

All commands run on the server. To access it:

1. Go to [console.hetzner.cloud](https://console.hetzner.cloud) and log in
2. Open the project → _Servers_ → `master-1`
3. Click the **>\_ Console** icon (top right) to open a web terminal
4. Log in as `root`

---

## 1. Is app data intact?

Open [https://balve.garmeres.com](https://balve.garmeres.com) and check if files, calendar, and contacts are present. Open [https://strapi.balve.garmeres.com/admin](https://strapi.balve.garmeres.com/admin) and check if blog posts and pages are present.

| Symptom                   | Action                                         |
| ------------------------- | ---------------------------------------------- |
| Nextcloud data is missing | → [Restore Nextcloud](02-restore-nextcloud.md) |
| Strapi data is missing    | → [Restore Strapi](03-restore-strapi.md)       |
| Both are fine             | Continue to step 2                             |

---

## 2. Are secrets working?

Check for pods that are not running:

```
kubectl get pods -A | grep -v Running | grep -v Completed
```

If pods are stuck on `CreateContainerConfigError`, inspect one:

```
kubectl describe pod <pod-name> -n <namespace>
```

If the error mentions a missing secret or secret key, the sealed-secrets signing key was lost.

| Symptom                                       | Action                                    |
| --------------------------------------------- | ----------------------------------------- |
| Pods failing with missing secret errors       | → [Re-seal secrets](04-reseal-secrets.md) |
| Pods are running or failing for other reasons | Continue to step 3                        |

---

## 3. Can you reach the cluster?

```
kubectl get nodes
```

| Symptom                                       | Action                                                                                                 |
| --------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| Server doesn't exist or `kubectl` fails       | → [Full rebuild](05-full-rebuild.md)                                                                   |
| Nodes are `Ready` but something else is wrong | Check ArgoCD at [argocd.balve.garmeres.com](https://argocd.balve.garmeres.com) for app-specific errors |
