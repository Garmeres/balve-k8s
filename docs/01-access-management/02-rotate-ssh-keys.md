# Rotate SSH keys

Replaces all SSH keys on `master-1` with your own key. Use this after revoking someone's access, or if you suspect the existing keys are compromised.

This does **not** affect the SSH key stored in the Hetzner project (under _Security_ → _SSH Keys_). That key is only used when creating **new** servers — it has no effect on running ones.

---

## Step 1: Generate a key pair

The Hetzner web console uses a US keyboard layout, which causes special characters like `+`, `/`, and `=` to be mangled when pasted. This command generates keys in a loop until it finds one without those characters, so the key can be safely pasted into the console:

```
while true; do rm -f ~/.ssh/hetzner-balve ~/.ssh/hetzner-balve.pub && ssh-keygen -t ed25519 -f ~/.ssh/hetzner-balve -N "" -C "hetzner-balve" -q && ! grep -q '[+/=]' ~/.ssh/hetzner-balve.pub && break; done
```

Print the public key and copy the entire line of output (it starts with `ssh-ed25519`):

```
cat ~/.ssh/hetzner-balve.pub
```

---

## Step 2: Open Hetzner Console

1. Log in to [console.hetzner.cloud](https://console.hetzner.cloud)
2. Open the project → _Servers_ → `master-1`
3. Click the **>\_ Console** icon (top right) to open a web terminal
4. Log in as `root`

---

## Step 3: Replace authorized keys

> **Keyboard layout:** The web console uses a US keyboard layout. If you're on a Norwegian/Nordic keyboard, most special characters are on different keys. The ones needed for the command below are:
>
> | Character | Key to press                                  |
> | --------- | --------------------------------------------- |
> | `>`       | Shift + the key just below Esc                |
> | `~`       | Shift + the key just left of Z                |
> | `/`       | The key just left of Right Shift              |
> | `_`       | Shift + `+` (the key between 0 and Backspace) |

1. Type this command and press Enter:
   > This command must be typed out, not pasted. See the keyboard layout mapping above to get the correct symbols.
   ```
   cat > ~/.ssh/authorized_keys
   ```
2. Paste the public key you copied in Step 1
3. Press **Enter**, then **Ctrl+D**

> **Note:** This overwrites the file. All previous keys are removed.

---

## Step 4: Configure local SSH and verify

Get the server IP from the Hetzner dashboard (go to _Servers_ → `master-1` — the IP is shown on the overview page). Then open `~/.ssh/config` in a text editor and add these lines, replacing `<master-1 IP>` with the actual IP:

```
Host balve-master
    HostName <master-1 IP>
    User root
    IdentityFile ~/.ssh/hetzner-balve
    IdentitiesOnly yes
```

If the file doesn't exist, create it.

Confirm you can connect:

```
ssh balve-master
```

---

## Step 5: Update Hetzner project key

Update the project-level SSH key so new servers get the correct key:

1. In [Hetzner Cloud Console](https://console.hetzner.cloud), go to the project → _Security_ → _SSH Keys_
2. Delete the old `hetzner-balve` key
3. Click _Add SSH Key_, paste the contents of `~/.ssh/hetzner-balve.pub`, and name it `hetzner-balve`
