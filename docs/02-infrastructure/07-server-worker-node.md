# Worker Node

In the Hetzner console, go to _Servers_ -> _Add Server_, and use the following values:

| Field            | Value                                                                                                               |
| ---------------- | ------------------------------------------------------------------------------------------------------------------- |
| Name             | `worker-1`                                                                                                          |
| Type             | `CPX42` (Shared resources, regular performance)                                                                     |
| Location         | `eu-central` (Helsinki)                                                                                             |
| Image            | `Ubuntu 24.04`                                                                                                      |
| Networking       | <ul><li>[x] Public IPv4</li><li>[ ] Public IPv6</li><li>[x] Private networks<ul><li>`network-1`</li></ul></li></ul> |
| SSH keys         | <ul><li>`hetzner-balve`</li></ul>                                                                                   |
| Volumes          |                                                                                                                     |
| Firewalls        | <ul><li>`firewall-2`</li></ul>                                                                                      |
| Backups          |                                                                                                                     |
| Placement groups | <ul><li>placement-group-1</li></ul>                                                                                 |
| Labels           |                                                                                                                     |
| Cloud config     | See [Cloud config](#cloud-config).                                                                                  |
| Server name      | `worker-1`                                                                                                          |

## Cloud config

Copy the entire [Cloud init script](#cloud-init-script), and paste it into the **Cloud Config** field of the Hetzner Server.

### SSH public key

Replace `<SSH PUBLIC KEY>` with the output of [Get SSH public key](./04-create-ssh-key.md#get-ssh-public-key), so that this:

```
ssh_authorized_keys:
  - <SSH PUBLIC KEY>
```

Becomes this:

```
ssh_authorized_keys:
  - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIB8... hetzner-balve
```

### Kubernetes join token

Replace `<JOIN TOKEN>` with the join token from the master node.

SSH into the master node by running:

```
ssh balve-master
```

Run the following command to print the token:

```
sudo cat /var/lib/rancher/k3s/server/node-token
```

It should print something like:

```
K1068ca...6::server:db1...67
```

Copy the entire line and use it to replace `<JOIN TOKEN>` in the script, so that this:

```
JOIN_TOKEN="<JOIN TOKEN>"
```

becomes this:

```
JOIN_TOKEN="K1068ca...6::server:db1...67"
```

### Cloud init script

```
#cloud-config
hostname: worker-1
manage_etc_hosts: true

package_update: true
package_upgrade: true

ssh_pwauth: false
disable_root: false

users:
  - name: root
    shell: /bin/bash
    lock_passwd: true
    ssh_authorized_keys:
      - <SSH PUBLIC KEY>

packages:
  - curl

write_files:
  - path: /etc/ssh/sshd_config.d/99-hardening.conf
    permissions: "0644"
    owner: root:root
    content: |
      PubkeyAuthentication yes
      PasswordAuthentication no
      KbdInteractiveAuthentication no
      ChallengeResponseAuthentication no
      AuthenticationMethods publickey
      PermitRootLogin prohibit-password
      UsePAM yes

runcmd:
  - |
      set -eux

      MASTER_IP="10.0.0.2"
      JOIN_TOKEN="<JOIN TOKEN>"

      PRIVATE_IP="$(ip route get ${MASTER_IP} | awk '/src/ {for (i=1;i<=NF;i++) if ($i=="src") print $(i+1)}' | head -n1)"
      PRIVATE_IFACE="$(ip route get ${MASTER_IP} | awk '/dev/ {for (i=1;i<=NF;i++) if ($i=="dev") print $(i+1)}' | head -n1)"
      test -n "${PRIVATE_IP}"
      test -n "${PRIVATE_IFACE}"

      mkdir -p /etc/rancher/k3s
      cat >/etc/rancher/k3s/config.yaml <<EOF
      node-name: worker-1
      node-ip: ${PRIVATE_IP}
      flannel-iface: ${PRIVATE_IFACE}
      server: https://${MASTER_IP}:6443
      token: ${JOIN_TOKEN}
      EOF

      mkdir -p /run/sshd
      sshd -t
      systemctl restart ssh || systemctl restart sshd

      until curl -kfsS "https://${MASTER_IP}:6443/ping" >/dev/null; do
        sleep 5
      done

      curl -sfL https://get.k3s.io | sh -s - agent

power_state:
  mode: reboot
  condition: true
```
