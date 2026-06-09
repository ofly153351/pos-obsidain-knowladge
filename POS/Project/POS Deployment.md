---
tags: [pos, deployment, server, pm2, cloudflare, tailscale, golive]
created: 2026-06-03
updated: 2026-06-08
---

# 🚀 POS Deployment

← [[POS Project Overview]] · [[POS Project MOC]]

---

## Server

Ubuntu 22.04 LTS — `root@ubuntu-2204`

## Public URLs

| Service | URL |
|---------|-----|
| Frontend | `https://pos.phiraphat.site` |
| Backend API | `https://api.phiraphat.site` |
| MinIO Storage | `https://media.phiraphat.site` |

---

## Process Manager: PM2

```bash
pm2 status               # ดู process ทั้งหมด
pm2 logs pos-backend     # log backend
pm2 logs pos-frontend    # log frontend
pm2 logs pos-tunnel      # log cloudflare tunnel
pm2 restart pos-backend  # restart หลัง git pull
pm2 monit                # CPU/RAM realtime
pm2 save                 # save process list (reboot survival)
```

PM2 ecosystem: `pos-backend/ecosystem.config.js`
- `pos-backend`: runs `./bin/api` (pre-built binary)
- `pos-frontend`: runs `npm start` (production build)

---

## golive.sh

`pos-backend/golive.sh` — one-command full startup:

```bash
bash golive.sh
```

**Steps:**
1. Pre-flight checks (docker, go, node, pm2, cloudflared, token file)
2. **Sync both repos to `origin/main`** (production deploys from **main**, not fix-of/dev)
3. `docker compose up -d` → wait PostgreSQL healthy
4. `go build -o bin/api cmd/api.go`
5. `rm -rf .next` → `npm install` → `npm run build`  ← clears stale Next cache
6. `pm2 start ecosystem.config.js`
7. `pm2 start cloudflared tunnel --config ~/.cloudflared/config.yml`
8. `pm2 save`
9. Health checks + print URLs

**Branch model:** dev work on a `fix-<person>/dev` branch → promote (`git merge --ff-only`) → `main` → push → production pulls `main`.
- `fix-of/dev` — primary dev branch (this session's work).
- `fix-kaew/dev` — Wutthichai's branch (frontend stock/inventory/barcode overhaul, 2026-06-09, not yet merged to main). See [[Barcode & Labels]] · [[Inventory & Stock Counting]].

**Git sync (updated 2026-06-08):** `sync_repo()` does
`fetch <tokenized-url> main` → `checkout -B main` → `reset --hard FETCH_HEAD`, dies on any failure.
Force-syncs to origin/main regardless of the server's current branch/dirty state.

**Token file setup:**
```bash
echo 'ghp_YOUR_NEW_TOKEN' > ~/.github_token   # classic PAT, scope: repo
chmod 600 ~/.github_token
```

### ⚠️ Git auth gotchas (hit in production)
- **Token must have `repo` scope** — without it, private-repo fetch fails with
  "Invalid username or token". Verify: `curl -H "Authorization: token $T" https://api.github.com/repos/ofly153351/pos-backend` → must return `full_name`.
- **Never persist the token in the remote URL.** Old golive set `remote set-url origin https://user:token@…`; when it died mid-fetch the URL was never restored, and the next run injected again → `https://user:token@user:token@…` → auth fail. Current golive builds a tokenized URL on the fly and `reset --hard FETCH_HEAD` (idempotent, strips existing creds first).
- **GitHub secret-scanning auto-revokes** any `ghp_…` token that passes through a monitored channel (e.g. pasted in chat). Always issue a fresh token and type it straight into the file on the server.
- Clean a stuck remote URL: `git remote set-url origin https://github.com/ofly153351/pos-backend.git`

---

## Docker Services

`docker-compose.yml` in `pos-backend/`:

| Service | Port |
|---------|------|
| PostgreSQL 15 | 5432 |
| MinIO API | 9000 |
| MinIO Console | 9001 |

```bash
docker compose up -d postgres minio minio-client
docker compose logs postgres
```

---

## Cloudflare Tunnel

Config: `~/.cloudflared/config.yml`

```yaml
tunnel: <tunnel-id>
credentials-file: /root/.cloudflared/<tunnel-id>.json

ingress:
  - hostname: pos.phiraphat.site
    service: http://localhost:3000
  - hostname: api.phiraphat.site
    service: http://localhost:8080
  - hostname: media.phiraphat.site
    service: http://localhost:9000
  - service: http_status:404
```

---

## MinIO .env Keys (Critical)

```bash
MINIO_ENDPOINT=127.0.0.1:9000      # SDK connection — internal
MINIO_USE_SSL=false                 # internal connection no SSL
MINIO_PUBLIC_URL=https://media.phiraphat.site  # NO bucket suffix!
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=<password>
MINIO_ROOT_USER=minioadmin
MINIO_ROOT_PASSWORD=<password>
MINIO_BUCKET_NAME=pos-assets
```

⚠️ `MINIO_PUBLIC_URL` must NOT end with `/pos-assets` — SDK adds bucket path automatically.

---

## Default Login

| Email | Password | Role |
|-------|----------|------|
| `owner@pos.dev` | `Owner1234!` | owner |
| `manager@pos.dev` | `Manager1234!` | manager |
| `cashier@pos.dev` | `Cashier1234!` | cashier |

---

## Navicat SSH Tunnel (DB access)

- SSH Host: server IP or Tailscale IP
- SSH Port: 22
- SSH User: root
- DB Host: 127.0.0.1, Port: 5432
- DB: pos_db, User: postgres

---

## Related
- [[POS Dev Commands]] — local dev commands
- [[Backend Architecture]] — migrations, env vars
