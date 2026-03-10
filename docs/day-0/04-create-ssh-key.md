# Create SSH-Key

Before we create the servers, we must define an SSH key that is allowed access to them. These steps are done on your local machine, and assumes MacOS or Linux.

Run the following in your machine's terminal:

```
ssh-keygen -t ed25519 -f ~/.ssh/hetzner-balve -C "hetzner-balve"
```

This creates the following files:

```
~/.ssh/hetzner-balve
~/.ssh/hetzner-balve.pub
```

Set the following permissions:

```
chmod 600 ~/.ssh/config
chmod 600 ~/.ssh/hetzner-balve
```

## Get SSH public key

Run the following command to get the public SSH key:

```
cat ~/.ssh/hetzner-balve.pub
```

You should get an output like this:

```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIB8... hetzner-balve
```

## Upload SSH key to Hetzner

In the Hetzner console, go to _Security_ -> _Add SSH key_. Set the fields as follows:

| Field              | Value                                                                |
| ------------------ | -------------------------------------------------------------------- |
| SSH key            | Copy and paste output from [Get SSH public key](#get-ssh-public-key) |
| Name               | `hetzner-balve`                                                      |
| Set as default key | Yes                                                                  |
