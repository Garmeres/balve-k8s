# ArgoCD

ArgoCD does not manage its own installation. To update, change `values.yaml` and re-run the install commands in the [deployment guide](../docs/03-deployment/01-deploy-argocd.md).

## RBAC

Users get access based on their GitHub or Nextcloud group membership:

| Group                 | Role        | Access                       |
| --------------------- | ----------- | ---------------------------- |
| `Garmeres:admins`     | `admin`     | Full access                  |
| `Garmeres:developers` | `developer` | View, sync, view logs        |
| `Garmeres board`      | `developer` | View, sync, view logs        |
| _(everyone else)_     | `viewer`    | View application status only |

> **Note:** The `developer` role cannot sync the `argocd` application itself. Only admins can do that.
