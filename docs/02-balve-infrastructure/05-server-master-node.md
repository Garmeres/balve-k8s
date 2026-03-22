# Master Node

In the Hetzner console, go to _Servers_ -> _Add Server_, and use the following values:

| Field            | Value                                                                                                               |
| ---------------- | ------------------------------------------------------------------------------------------------------------------- |
| Name             | `master-1`                                                                                                          |
| Type             | `CPX22` (Shared resources, regular performance)                                                                     |
| Location         | `eu-central` (Helsinki)                                                                                             |
| Image            | `Ubuntu 24.04`                                                                                                      |
| Networking       | <ul><li>[x] Public IPv4</li><li>[ ] Public IPv6</li><li>[x] Private networks<ul><li>`network-1`</li></ul></li></ul> |
| SSH keys         | `hetzner-balve` (see [SSH keys](04-ssh-keys.md))                                                                    |
| Volumes          |                                                                                                                     |
| Firewalls        | <ul><li>`firewall-1`</li></ul>                                                                                      |
| Backups          |                                                                                                                     |
| Placement groups | <ul><li>placement-group-1</li></ul>                                                                                 |
| Labels           |                                                                                                                     |
| Cloud config     | See [Cloud config](#cloud-config).                                                                                  |
| Server name      | `master-1`                                                                                                          |

## Cloud config

Copy the entire [Cloud init script](#cloud-init-script), and paste it into the **Cloud Config** field of the Hetzner Server.

### Join token

Before pasting the script, generate a join token on your local machine:

```
openssl rand -hex 32
```

Replace `<TOKEN>` in the `JOIN_TOKEN` variable near the top of the script with this value. Save the token — you will need it again for the [worker node](06-server-worker-node.md).

### Cloud init script

```
#cloud-config
hostname: master-1
manage_etc_hosts: true

package_update: true
package_upgrade: true

users:
  - name: root
    shell: /bin/bash

packages:
  - curl
  - git
  - awscli

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

      JOIN_TOKEN="<TOKEN>"

      PRIVATE_IP="$(ip route get 10.0.0.3 | awk '/src/ {for (i=1;i<=NF;i++) if ($i=="src") print $(i+1)}' | head -n1)"
      PRIVATE_IFACE="$(ip route get 10.0.0.3 | awk '/dev/ {for (i=1;i<=NF;i++) if ($i=="dev") print $(i+1)}' | head -n1)"
      test -n "${PRIVATE_IP}"
      test -n "${PRIVATE_IFACE}"

      mkdir -p /etc/rancher/k3s
      cat >/etc/rancher/k3s/config.yaml <<EOF
      node-name: master-1
      token: ${JOIN_TOKEN}
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

      curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

      KUBESEAL_VERSION=$(curl -s https://api.github.com/repos/bitnami-labs/sealed-secrets/releases/latest | grep '"tag_name"' | sed 's/.*"v\(.*\)".*/\1/')
      curl -OL "https://github.com/bitnami-labs/sealed-secrets/releases/download/v${KUBESEAL_VERSION}/kubeseal-${KUBESEAL_VERSION}-linux-amd64.tar.gz"
      tar -xzf kubeseal-${KUBESEAL_VERSION}-linux-amd64.tar.gz kubeseal
      install -m 755 kubeseal /usr/local/bin/kubeseal
      rm kubeseal kubeseal-${KUBESEAL_VERSION}-linux-amd64.tar.gz

power_state:
  mode: reboot
  condition: true
```

## Configure SSH

After the server is created, copy its public IP from the Hetzner Console and add the following to `~/.ssh/config` on your local machine:

```
Host balve-master
    HostName <master-1 public IP>
    User root
    IdentityFile ~/.ssh/hetzner-balve
    IdentitiesOnly yes
```

Replace `<master-1 public IP>` with the actual IP. After this, you can connect with `ssh balve-master`.
