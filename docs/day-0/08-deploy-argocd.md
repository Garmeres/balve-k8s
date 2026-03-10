# Deploy ArgoCD

## Install

From your local machine, in the root of the repo:

```
cd argo-cd
helm dependency build
helm template argocd . -n argocd | ssh balve-master 'kubectl apply -n argocd --server-side -f -'
```

## Verify the installation

SSH into the master node:

```
ssh balve-master
```

Wait for all pods to become ready:

```
kubectl get pods -n argocd -w
```

## Get the admin password

```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 -d && echo
```

## Access the ArgoCD UI

On the master node, start port-forwarding:

```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

In another terminal on your local machine, open an SSH tunnel:

```
ssh -L 8080:localhost:8080 balve-master
```

Open [https://localhost:8080](https://localhost:8080) in your browser. Log in with username `admin` and the password from the previous step.

ArgoCD will pick up the ApplicationSet and begin syncing all applications defined under `applications/`.
