# Board member becomes technical manager

The technical manager (sysadmin) is responsible for the Balve platform — the servers, deployments, and code. This assumes the person is already a board member with a Balve (Nextcloud) account, a garmeres email, and a Strapi account.

1. [Make them an admin](09-manage-balve-users.md#add-a-user-to-a-group) in Balve by adding them to the **admin** group. This gives them full control over Nextcloud — managing users, apps, and settings
2. [Add them to the **admin@garmeres.com** forwarding group](10-manage-email-users.md#add-a-user-to-a-forwarding-group). This is the email group for the chairman, deputy chairman, technical manager, and other elevated roles
3. [Make them a **Super Admin**](11-manage-strapi-users.md#change-a-users-role) in Strapi. This gives them full control over the CMS, including inviting and removing other users
4. [Add them to the Garmeres GitHub organization](12-manage-github-users.md#add-a-user) and [place them in the **admins** team](12-manage-github-users.md#add-a-user-to-a-team). This gives them access to all code repositories with admin permissions
5. [Add them to the Hetzner project](13-manage-hetzner-users.md#add-a-user). This gives them access to the servers and cloud infrastructure where the platform runs
