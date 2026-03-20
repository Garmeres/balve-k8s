# Handover

When the teknisk ansvarlig role changes, follow these steps in order. The outgoing person and the incoming person must work together.

## Step 1: Incoming person creates accounts

1. Create a Hetzner Cloud account at [console.hetzner.cloud](https://console.hetzner.cloud)
2. Create a GitHub account at [github.com](https://github.com) (if they don't already have one)

## Step 2: Outgoing person grants access

Do each of these with the incoming person:

### Root

1. Log in to [Domeneshop](https://domene.shop/login) with the user email **admin@garmeres.com**
2. Go to **Mine domener** → **garmeres.com** → **Epost**
3. If the incoming person doesn't already have a `@garmeres.com` email, create one (e.g. `fornavn@garmeres.com`) that forwards to their personal email
4. Find the address **admin@garmeres.com** and click it
5. Click **Legg til flere mottakere**
6. Enter the incoming person's new `@garmeres.com` address
7. Click **Legg til**

### Infrastructure

1. Log in to [Hetzner Cloud Console](https://console.hetzner.cloud)
2. Open the project **balve** (ID 12640153)
3. Go to **Members** → **Add member**
4. Enter the incoming person's email address, set the role to **Admin**, and click **Add**
5. Share the Domeneshop password from Balve passwords
6. Go to [github.com/orgs/Garmeres/people](https://github.com/orgs/Garmeres/people), click **Invite member**, and invite the incoming person

### Application admin

1. In Nextcloud at [balve.garmeres.com](https://balve.garmeres.com), go to **Settings** → **Users**. If the incoming person doesn't have an account, create one with an empty password and click **Send welcome email**. Add them to the **admin** group.
2. In Strapi at [strapi.balve.garmeres.com/admin](https://strapi.balve.garmeres.com/admin), go to **Settings** → **Administration panel**, click **Invite new user**, and invite the incoming person with the **admin** role

## Step 3: Verify

The incoming person should verify they can:

1. Receive email sent to `admin@garmeres.com`
2. Log in to [Hetzner Cloud Console](https://console.hetzner.cloud) and see the **balve** project
3. Log in to [Domeneshop](https://domene.shop/login)
4. Access the [Garmeres GitHub organization](https://github.com/Garmeres)
5. Log in to Nextcloud as an admin
6. Log in to Strapi as an admin

## Step 4: Incoming person revokes outgoing person's access

### Root

1. Log in to [Domeneshop](https://domene.shop/login) with the user email **admin@garmeres.com**
2. Go to **Mine domener** → **garmeres.com** → **Epost**
3. Find the address **admin@garmeres.com** and click it
4. Find the outgoing person's email address in the recipient list and remove it

### Infrastructure

1. In [Hetzner Cloud Console](https://console.hetzner.cloud), open the project **balve**, go to **Members**, find the outgoing person, and click **Remove**
2. Change the Domeneshop password in [Domeneshop](https://domene.shop/login) and update it in the Balve passwords app in Nextcloud
3. Go to [github.com/orgs/Garmeres/people](https://github.com/orgs/Garmeres/people), find the outgoing person, and remove them from the organization

### Application admin

1. In Nextcloud, go to **Settings** → **Users**, find the outgoing person, and remove them from the **admin** group
2. In Strapi, go to **Settings** → **Administration panel**, find the outgoing person, and change their role or delete their account
