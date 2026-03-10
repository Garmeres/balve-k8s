# SSH config for master-1

Find the public IP address of the master node. In the Hetzner console, go to _Servers_, and copy the public IP of the master node.

Create or edit:

```
nano ~/.ssh/config
```

Add

```
Host balve-master
    HostName <MASTER_PUBLIC_IP>
    User root
    IdentityFile ~/.ssh/hetzner-balve
    IdentitiesOnly yes
```

But replace `<MASTER_PUBLIC_IP>` with the public IP address of the master node, so that this:

```
Host balve-master
    HostName <MASTER_PUBLIC_IP>
```

becomes something like this:

```
Host balve-master
    HostName 12.345.67.89
```

Then you should be able to ssh into the master and worker nodes, simply by running

```
ssh balve-master
```
