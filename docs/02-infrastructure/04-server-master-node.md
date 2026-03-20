# Master Node

In the Hetzner console, go to _Servers_ -> _Add Server_, and use the following values:

| Field            | Value                                                                                                               |
| ---------------- | ------------------------------------------------------------------------------------------------------------------- |
| Name             | `master-1`                                                                                                          |
| Type             | `CPX22` (Shared resources, regular performance)                                                                     |
| Location         | `eu-central` (Helsinki)                                                                                             |
| Image            | `Ubuntu 24.04`                                                                                                      |
| Networking       | <ul><li>[x] Public IPv4</li><li>[ ] Public IPv6</li><li>[x] Private networks<ul><li>`network-1`</li></ul></li></ul> |
| SSH keys         | <ul><li>`hetzner-balve`</li></ul>                                                                                   |
| Volumes          |                                                                                                                     |
| Firewalls        | <ul><li>`firewall-1`</li></ul>                                                                                      |
| Backups          |                                                                                                                     |
| Placement groups | <ul><li>placement-group-1</li></ul>                                                                                 |
| Labels           |                                                                                                                     |
| Cloud config     | See [Cloud config](#cloud-config).                                                                                  |
| Server name      | `master-1`                                                                                                          |

## Cloud config

Copy the entire [Cloud init script](#cloud-init-script), and paste it into the **Cloud Config** field of the Hetzner Server. Replace `<SSH PUBLIC KEY>` with the output of [Get SSH public key](./04-create-ssh-key.md#get-ssh-public-key), so that this:

```
ssh_authorized_keys:
  - <SSH PUBLIC KEY>
```

Becomes this:

```
ssh_authorized_keys:
  - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIB8... hetzner-balve
```

### Cloud init script

```
#cloud-config
hostname: master-1
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

  - path: /var/lib/rancher/k3s/server/manifests/traefik-config.yaml
    permissions: "0644"
    owner: root:root
    content: |
      apiVersion: helm.cattle.io/v1
      kind: HelmChartConfig
      metadata:
        name: traefik
        namespace: kube-system
      spec:
        valuesContent: |-
          additionalArguments:
            - "--entryPoints.web.http.redirections.entryPoint.to=:443"
            - "--entryPoints.web.http.redirections.entryPoint.scheme=https"

runcmd:
  - |
      set -eux

      PRIVATE_IP="$(ip route get 10.0.0.3 | awk '/src/ {for (i=1;i<=NF;i++) if ($i=="src") print $(i+1)}' | head -n1)"
      PRIVATE_IFACE="$(ip route get 10.0.0.3 | awk '/dev/ {for (i=1;i<=NF;i++) if ($i=="dev") print $(i+1)}' | head -n1)"
      test -n "${PRIVATE_IP}"
      test -n "${PRIVATE_IFACE}"

      mkdir -p /etc/rancher/k3s
      cat >/etc/rancher/k3s/config.yaml <<EOF
      node-name: master-1
      write-kubeconfig-mode: "0644"
      node-ip: ${PRIVATE_IP}
      advertise-address: ${PRIVATE_IP}
      flannel-iface: ${PRIVATE_IFACE}
      tls-san:
        - ${PRIVATE_IP}
      EOF

      mkdir -p /run/sshd
      sshd -t
      systemctl restart ssh || systemctl restart sshd

      curl -sfL https://get.k3s.io | sh -s - server

power_state:
  mode: reboot
  condition: true
```
