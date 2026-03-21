# Firewalls

For each firewall listed below, go to _Firewalls_ -> _Create firewall_ in the Hetzner console, and use the values listed below:

## firewall-1

**Name:** `firewall-1`

**Inbound rules:**

| Source      | Protocol | Port  |
| ----------- | -------- | ----- |
| Any IPv4    | TCP      | 22    |
| Any IPv4    | ICMP     | -     |
| 10.0.0.0/24 | TCP      | 6443  |
| 10.0.0.0/24 | TCP      | 10250 |
| 10.0.0.0/24 | UDP      | 8472  |

**Outbound rules:**

Allow all

## firewall-2

**Name:** `firewall-2`

**Inbound rules:**

| Source      | Protocol | Port  |
| ----------- | -------- | ----- |
| Any IPv4    | TCP      | 22    |
| 10.0.0.0/24 | UDP      | 8472  |
| 10.0.0.0/24 | TCP      | 10250 |
| Any IPv4    | TCP      | 80    |
| Any IPv4    | TCP      | 443   |

**Outbound rules:**

Allow all
