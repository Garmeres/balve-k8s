# Deploy ArgoCD

All commands in this document are run on `master-1` via the Hetzner web console.

## GitHub Teams

Go to [github.com/orgs/garmeres/teams](https://github.com/orgs/garmeres/teams) and create:

- `admins` — full ArgoCD access
- `developers` — read + sync access

## Clone the repo

```
git clone https://github.com/Garmeres/balve-k8s.git
cd balve-k8s
```

## Install ArgoCD

```
helm dependency build argo-cd
helm template argocd argo-cd -n argocd | kubectl apply -n argocd --server-side -f -
```

> **Note:** ArgoCD does not reconcile itself. Any changes to `argo-cd/values.yaml` must be reapplied by pulling the latest code and re-running the command above.

## Verify the installation

Wait for all pods to become ready:

```
kubectl get pods -n argocd -w
```

## Apply the ApplicationSet

Once all ArgoCD pods are running, bootstrap the `argocd-config` application (which contains the ApplicationSet):

```
helm template argocd-config applications/argocd-config -n argocd \
  --show-only templates/applicationset.yaml \
  | kubectl apply -n argocd --server-side -f -
```

After this initial apply, ArgoCD manages `argocd-config` like any other application — future changes are synced automatically.

## Access the ArgoCD UI

ArgoCD will pick up the ApplicationSet and begin syncing all applications. Wait for cert-manager to finish syncing:

```
kubectl get applications -n argocd
```

Once it shows `Synced` and `Healthy`, open [https://argocd.balve.garmeres.com](https://argocd.balve.garmeres.com). GitHub login will not work until the sealed secrets are created in the [next step](02-create-sealed-secrets.md).

## RBAC

All users get read-only access (role `viewer`) by default. The following groups have elevated access via GitHub or Nextcloud (see [03-nextcloud-oidc.md](03-nextcloud-oidc.md)):

| Group                 | Role        | Access                       |
| --------------------- | ----------- | ---------------------------- |
| `Garmeres:admins`     | `admin`     | Full access                  |
| `Garmeres:developers` | `developer` | View, sync, view logs        |
| `Technical`           | `developer` | View, sync, view logs        |
| _(everyone else)_     | `viewer`    | View application status only |
