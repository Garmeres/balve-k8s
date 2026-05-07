# DNS Records

In the [Domeneshop control panel](https://domene.shop/login), log in with **admin@garmeres.com**. Go to _Mine domener_ → _garmeres.com_ → _DNS_.

Create two A records pointing to the **worker node's public IPv4 address** (found in the Hetzner console under _Servers_ → `worker-1`).

Only the worker node has ports 80/443 open (via `firewall-2`).

| Host      | Type | Value              |
| --------- | ---- | ------------------ |
| `balve`   | A    | _worker public IP_ |
| `*.balve` | A    | _worker public IP_ |

## Email (SPF)

For the Maddy mail relay to pass SPF checks, the domain's SPF record must include the worker node. Verify that the existing SPF TXT record on `@` contains:

```
include:balve.garmeres.com
```

If it doesn't, add it. For example: `v=spf1 ... include:balve.garmeres.com ~all`
