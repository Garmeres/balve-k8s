# Medama Analytics

Privacy-focused, cookie-free website analytics. Single binary with embedded DuckDB — no external database required.

Default login: `admin` / `CHANGE_ME_ON_FIRST_LOGIN`. Change immediately after first login.

## Tracking snippet

Add to garmeres-frontend layout `<head>`:

```html
<script defer src="https://analytics.balve.garmeres.com/script.js"></script>
```

## Backup

The analytics data is stored in two DuckDB files on the PVC:
- `me_meta.db` — user accounts and site configuration
- `me_analytics.db` — all analytics data

Back these up periodically. A simple approach:

```bash
kubectl cp medama/<pod-name>:/data/me_meta.db ./me_meta.db
kubectl cp medama/<pod-name>:/data/me_analytics.db ./me_analytics.db
```
