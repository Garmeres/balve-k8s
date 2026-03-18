# DNS Records

At your DNS provider, create A records pointing to the **worker node's public IPv4 address**.

Only the worker node has ports 80/443 open (via `firewall-2`).

| Hostname                | Type | Value              |
| ----------------------- | ---- | ------------------ |
| `dev.garmeres.com`      | A    | _worker public IP_ |
| `edit.dev.garmeres.com` | A    | _worker public IP_ |
| `*.dev.garmeres.com`    | A    | _worker public IP_ |
