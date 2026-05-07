# DNS Records

In the [Domeneshop control panel](https://domene.shop/login), log in with **admin@garmeres.com**. Go to _Mine domener_ → _garmeres.com_ → _DNS_.

Create two A records pointing to the **worker node's public IPv4 address** (found in the Hetzner console under _Servers_ → `worker-1`).

Only the worker node has ports 80/443 open (via `firewall-2`).

| Host      | Type | Value              |
| --------- | ---- | ------------------ |
| `balve`   | A    | _worker public IP_ |
| `*.balve` | A    | _worker public IP_ |

## Email (SPF)

The Maddy mail relay sends email from `balve.garmeres.com`. Two DNS changes are needed for SPF to pass:

1. **Edit** the existing TXT record on `@` and add `include:balve.garmeres.com` to the SPF value (before the `~all` or `-all` at the end).

2. **Create** a new TXT record:

| Host    | Type | Value           |
| ------- | ---- | --------------- |
| `balve` | TXT  | `v=spf1 a -all` |
