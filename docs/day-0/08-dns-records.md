# DNS Records

At your DNS provider, create A records pointing to the **worker node's public IPv4 address**.

Only the worker node has ports 80/443 open (via `firewall-2`).

| Hostname               | Type | Value              |
| ---------------------- | ---- | ------------------ |
| `balve.garmeres.com`   | A    | _worker public IP_ |
| `*.balve.garmeres.com` | A    | _worker public IP_ |
