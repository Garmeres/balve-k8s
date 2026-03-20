# Grant access

## 1. Root

### Admin email

1. Log in to [Domeneshop](https://domene.shop/login) with the user email **admin@garmeres.com**
2. Go to **Mine domener** → **garmeres.com** → **Epost**
3. Find the address **admin@garmeres.com** and click it
4. Click **Legg til flere mottakere**
5. Enter the person's email address
6. Click **Legg til**

### GitHub

1. Go to [github.com/orgs/Garmeres/people](https://github.com/orgs/Garmeres/people)
2. Click **Invite member**
3. Enter the person's GitHub username or email
4. Send the invitation

The person will need to accept the invitation.

## 2. Infrastructure

### Hetzner Cloud

1. Log in to [Hetzner Cloud Console](https://console.hetzner.cloud)
2. Open the project **balve** (ID 12640153)
3. Go to **Members** in the left sidebar
4. Click **Add member**
5. Enter the person's email address
6. Set the role to **Admin**
7. Click **Add**

The person will receive an email invitation. If they don't already have a Hetzner account, they will need to create one.

### Domeneshop

Share the login credentials from Balve passwords with the person.

## 3. Application superadmin

These are shared accounts — no granting needed. See [overview](01-overview.md) for credentials.

## 4. Admin user

### Nextcloud

1. Log in to [balve.garmeres.com](https://balve.garmeres.com) as an admin
2. Go to **Settings** → **Users**
3. Find the user and add them to the **admin** group

### Strapi

1. Log in to [strapi.balve.garmeres.com/admin](https://strapi.balve.garmeres.com/admin) as an admin
2. Go to **Settings** → **Administration panel**
3. Find the user and assign the **admin** role

## 5. User

### Garmeres email

1. Log in to [Domeneshop](https://domene.shop/login) with the user email **admin@garmeres.com**
2. Go to **Mine domener** → **garmeres.com** → **Epost**
3. Create a new email address for the person (e.g. `fornavn@garmeres.com`) that forwards to their personal email

### Nextcloud

1. Log in to [balve.garmeres.com](https://balve.garmeres.com) as an admin
2. Go to **Settings** → **Users**
3. Click **New user**
4. Fill in username and display name, leave the password empty
5. Click **Send welcome email**
6. Click **Add a new user**

### Strapi

1. Log in to [strapi.balve.garmeres.com/admin](https://strapi.balve.garmeres.com/admin) as an admin
2. Go to **Settings** → **Administration panel**
3. Click **Invite new user**
4. Fill in email, first name, last name, and role
5. Click **Invite**
