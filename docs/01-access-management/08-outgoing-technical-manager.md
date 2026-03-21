# Outgoing technical manager

The outgoing technical manager had admin access to everything. Several services need to be cleaned up, and the extent depends on whether they are remaining on the board or leaving entirely.

1. [Demote their Balve user](09-manage-balve-users.md#remove-a-user-from-a-group) by removing them from the **admin** group. If they are still on the board, keep them in **Board members**. If they are also leaving the board, demote them to **Everyone**
2. [Remove them from the **admin@garmeres.com** forwarding group](10-manage-email-users.md#remove-a-user-from-a-forwarding-group) — they should no longer receive admin mail
3. If they are also leaving the board, [remove them from **meile@garmeres.com**](10-manage-email-users.md#remove-a-user-from-a-forwarding-group) as well. Ask if it is fine to [delete their garmeres email](10-manage-email-users.md#remove-an-email-address) completely — we only have 20 addresses
4. [Demote their Strapi account](11-manage-strapi-users.md#change-a-users-role). If they remain on the board, demote to **Editor** so they can still edit all website content. If they are not remaining on the board, ask if they need continued access — if yes, demote to **Author** (can only edit their own content); if no, [delete their account](11-manage-strapi-users.md#remove-a-user)
5. Ask if they wish to remain a collaborator on the Balve platform code. If yes, [remove them from the **admins** GitHub team](12-manage-github-users.md#remove-a-user-from-a-team) and [add them to **developers**](12-manage-github-users.md#add-a-user-to-a-team) instead. If no, [remove them from the Garmeres organization](12-manage-github-users.md#remove-a-user) entirely
6. [Remove their Hetzner user](13-manage-hetzner-users.md#remove-a-user) from the project — they should no longer have access to the servers
7. [Rotate the SSH key for master-1](02-rotate-ssh-keys.md). **Do not skip this step.** The outgoing technical manager had the private key that grants root access to the server. If you do not rotate it, they can still log in and control the entire platform at any time
