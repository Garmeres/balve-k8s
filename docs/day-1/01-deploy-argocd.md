# Deploy ArgoCD

## GitHub Teams

Go to _github.com/orgs/garmeres/teams_ and create:

- `admins` — full ArgoCD access
- `developers` — read-only ArgoCD access

## GitHub OAuth App

Go to _github.com/organizations/garmeres/settings/applications_ -> _New OAuth App_ and fill in:

| Field                      | Value                                              |
| -------------------------- | -------------------------------------------------- |
| Application name           | `ArgoCD`                                           |
| Homepage URL               | `https://argocd.dev.garmeres.com`                  |
| Authorization callback URL | `https://argocd.dev.garmeres.com/api/dex/callback` |
| Enable Device Flow         | checked                                            |

Click _Register application_.

On the next page, the **Client ID** is shown near the top. Click _Generate a new client secret_ — the secret is only shown once, copy it immediately.

Open a terminal and run these two lines, replacing the values with what you just copied:

```
GITHUB_CLIENT_ID="<Client ID>"
GITHUB_CLIENT_SECRET="<Client Secret>"
```

Keep this terminal open — the install step below uses these variables.

## Install

From your local machine, in the root of the repo:

```
cd argo-cd
helm dependency build
helm template argocd . -n argocd \
  --set argo-cd.configs.secret.extra."dex\.github\.clientID"="$GITHUB_CLIENT_ID" \
  --set argo-cd.configs.secret.extra."dex\.github\.clientSecret"="$GITHUB_CLIENT_SECRET" \
  | ssh balve-master 'kubectl apply -n argocd --server-side -f -'
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

## Apply the ApplicationSet

Once all ArgoCD pods are running, apply the ApplicationSet from your local machine:

```
cat applicationset.yaml | ssh balve-master 'kubectl apply -f -'
```

## Access the ArgoCD UI

ArgoCD will pick up the ApplicationSet and begin syncing all applications. Wait for cert-manager to finish syncing:

```
kubectl get applications -n argocd
```

Once it shows `Synced` and `Healthy`, open [https://argocd.dev.garmeres.com](https://argocd.dev.garmeres.com) and log in with GitHub.
