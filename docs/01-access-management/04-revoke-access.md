# Revoke access

## 1. Root

### Admin email

1. Log in to [Domeneshop](https://domene.shop/login) with the user email **admin@garmeres.com**
2. Go to **Mine domener** → **garmeres.com** → **Epost**
3. Find the address **admin@garmeres.com** and click it
4. Find the person's email address in the recipient list and remove it

### GitHub

1. Go to [github.com/orgs/Garmeres/people](https://github.com/orgs/Garmeres/people)
2. Find the person
3. Click the gear icon next to their name and select **Remove from organization**

## 2. Infrastructure

### Hetzner Cloud

1. Log in to [Hetzner Cloud Console](https://console.hetzner.cloud)
2. Open the project **balve** (ID 12640153)
3. Go to **Members** in the left sidebar
4. Find the person and click **Remove**

### Domeneshop

Change the password in [Domeneshop](https://domene.shop/login) and update it in Balve passwords.

## 3. Application superadmin

These are shared accounts. If compromised, reset the passwords:

- **Nextcloud:** Reset via `kubectl` on the server (see [disaster recovery](../04-disaster-recovery/02-restore-nextcloud.md))
- **Strapi:** Use forgot password at [strapi.balve.garmeres.com/admin](https://strapi.balve.garmeres.com/admin) — reset goes to `admin@garmeres.com`

## 4. Admin user

### Nextcloud

1. Log in to [balve.garmeres.com](https://balve.garmeres.com) as an admin
2. Go to **Settings** → **Users**
3. Find the user and remove them from the **admin** group

### Strapi

Requires a Super Admin (level 3).

1. Log in to [strapi.balve.garmeres.com/admin](https://strapi.balve.garmeres.com/admin) as a Super Admin
2. Go to **Settings** → **Administration panel**
3. Find the user and change their role, or delete their account

## 5. User

### Nextcloud

1. Log in to [balve.garmeres.com](https://balve.garmeres.com) as an admin
2. Go to **Settings** → **Users**
3. Find the user and click **Delete user** or **Disable user**

### Strapi

1. Log in to [strapi.balve.garmeres.com/admin](https://strapi.balve.garmeres.com/admin) as an admin
2. Go to **Settings** → **Administration panel**
3. Find the user and click **Delete**
