# Outgoing board member

1. [Remove them from the **Board members** group](09-manage-balve-users.md#remove-a-user-from-a-group) in Balve (Nextcloud). This demotes them to **Everyone**, meaning they lose access to the **Garmeres board's internal files** shared folder but keep access to **Shared files for all Balve users** and their personal files. They can delete their own account later if they want to
2. Ask if you may delete their garmeres email address — we only have 20 addresses on the plan, so freeing one up is helpful
3. [Remove them from all email forwarding groups](10-manage-email-users.md#remove-a-user-from-a-forwarding-group) — **meile@garmeres.com** and **admin@garmeres.com** (if they were in it). Do this even if you already deleted the address — a deleted address still appears in forwarding groups and will cause bounce errors if not removed
4. If they agreed to delete the address, [delete it](10-manage-email-users.md#remove-an-email-address)
5. Ask if they want to remain an author on the Garmeres website. If yes, [demote their Strapi user](11-manage-strapi-users.md#change-a-users-role) to **Author** — they will only be able to edit their own content. If no, [delete their Strapi account](11-manage-strapi-users.md#remove-a-user)
