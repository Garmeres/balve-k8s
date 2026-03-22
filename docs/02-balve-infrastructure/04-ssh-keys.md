# SSH Keys

In the [Hetzner Cloud Console](https://console.hetzner.cloud), go to the project **balve** → **Security** → **SSH Keys** → **Add SSH Key**.

### Generate a key

On your local machine, open a terminal and run:

```
ssh-keygen -t ed25519 -f ~/.ssh/hetzner-balve
```

Press Enter twice when asked for a passphrase (leave it empty).

Then copy the public key:

```
cat ~/.ssh/hetzner-balve.pub
```

Paste the output into the Hetzner SSH key form. Set the name to `hetzner-balve`.
