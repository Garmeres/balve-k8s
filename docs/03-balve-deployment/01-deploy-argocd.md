# Deploy ArgoCD

All commands in this document are run **on your local machine**, from the root of the `balve-k8s` repo.

Wait for the cloud-init script to finish before starting:

```
ssh balve-master "cloud-init status --wait"
```

## 1. Create the namespace

```
ssh balve-master "kubectl apply --server-side -f -" < argo-cd/templates/namespace.yaml
```

## 2. Install ArgoCD

```
helm dependency build argo-cd
helm template argocd argo-cd -n argocd | ssh balve-master "kubectl apply -n argocd --server-side -f -"
```

This installs ArgoCD into the cluster. Wait for all pods to become ready:

```
ssh balve-master "kubectl get pods -n argocd -w"
```

All pods should show `Running` or `Completed`. Press `Ctrl+C` to stop watching.

> **Note:** ArgoCD does not manage its own installation. If you later change `argo-cd/values.yaml`, you must re-run the commands above.

## 3. Apply the ApplicationSet

This tells ArgoCD about all the applications it should manage:

```
helm template argocd-config applications/argocd-config -n argocd \
  --show-only templates/applicationset.yaml \
  | ssh balve-master "kubectl apply -n argocd --server-side -f -"
```

ArgoCD will start syncing applications automatically. After this initial apply, ArgoCD manages `argocd-config` like any other application — future changes are synced from git.

## 4. Force sync argocd-config

On a fresh install, ArgoCD needs a one-time forced sync of the `argocd-config` application so it applies the correct annotations to all its child resources. This will show as `OutOfSync` until the [sealed secrets](02-create-sealed-secrets.md) are created in the next step — that is expected.

```
ssh balve-master "kubectl patch application argocd-config -n argocd --type merge -p '{\"operation\":{\"sync\":{\"syncStrategy\":{\"apply\":{\"force\":true}}}}}'"
```

## 5. Verify

Check that all applications are syncing:

```
ssh balve-master "kubectl get applications -n argocd"
```

Wait for `cert-manager` to show `Synced` and `Healthy` — this is needed for HTTPS certificates. Other applications will show errors until the [sealed secrets](02-create-sealed-secrets.md) are created in the next step.

Once cert-manager is healthy, the ArgoCD dashboard is available at [https://argocd.balve.garmeres.com](https://argocd.balve.garmeres.com).

> **Note:** You cannot log in to ArgoCD yet. Login requires GitHub OAuth, which depends on the sealed secrets created in the [next step](02-create-sealed-secrets.md). The admin password is disabled.
