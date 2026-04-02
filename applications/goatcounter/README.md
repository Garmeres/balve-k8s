# GoatCounter

Privacy-focused, cookie-free website analytics. Single Go binary with embedded SQLite — no external dependencies, no runtime downloads.

## First-time setup

After deployment, open https://analytics.balve.garmeres.com and follow the setup wizard to create the first site and admin account.

Alternatively via CLI:

```bash
kubectl exec -it -n goatcounter deploy/goatcounter -- \
  goatcounter db create site -vhost=analytics.balve.garmeres.com -user.email=<your-email>
```

## Tracking snippet

Add to garmeres-frontend layout `<head>`:

```html
<script data-goatcounter="https://analytics.balve.garmeres.com/count"
        async src="https://analytics.balve.garmeres.com/count.js"></script>
```

## Backup

The SQLite database is stored on the PVC at `/home/goatcounter/goatcounter-data/db.sqlite3`.

```bash
kubectl cp goatcounter/<pod-name>:/home/goatcounter/goatcounter-data/db.sqlite3 ./goatcounter-backup.sqlite3
```
