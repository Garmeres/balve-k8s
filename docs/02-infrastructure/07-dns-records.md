# DNS Records

In the [Domeneshop control panel](https://domene.shop/login), log in with **admin@garmeres.com**. Go to _Mine domener_ → _garmeres.com_ → _DNS_.

Create two A records pointing to the **worker node's public IPv4 address** (found in the Hetzner console under _Servers_ → `worker-1`).

Only the worker node has ports 80/443 open (via `firewall-2`).

| Host      | Type | Value              |
| --------- | ---- | ------------------ |
| `balve`   | A    | _worker public IP_ |
| `*.balve` | A    | _worker public IP_ |
