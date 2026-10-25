# TorrentDrive

Self-hosted torrent-to-cloud platform. Upload `.torrent` files, inspect live swarm health, pick the
exact files you want, and stream them straight to Google Drive or any S3-compatible storage with
automatic failover.

One container. Zero configuration files.

## Install in One Command

### Linux / macOS

<!-- snippet:install-linux -->
```bash
curl -fsSL https://raw.githubusercontent.com/TorrentDrive/deploy/main/install.sh | bash
```
<!-- /snippet -->

### Docker Compose (Zero `.env` Needed)

```bash
curl -fsSL https://raw.githubusercontent.com/TorrentDrive/deploy/main/docker-compose.yml -o docker-compose.yml
docker compose up -d
```

### Windows (PowerShell)

<!-- snippet:install-windows -->
```powershell
irm https://raw.githubusercontent.com/TorrentDrive/deploy/main/install.ps1 | iex
```
<!-- /snippet -->

### 1-Click PaaS Blueprints

| Platform | Blueprint Path | Highlights |
|----------|----------------|------------|
| **Dokploy** | [`paas/dokploy/`](./paas/dokploy) | Includes `docker-compose.yml` + native `template.toml` for Traefik port `47100` |
| **Coolify** | [`paas/coolify/`](./paas/coolify) | Uses `SERVICE_FQDN_TORRENTDRIVE_47100` & `SERVICE_URL_TORRENTDRIVE` magic variables |
| **Render** | [`paas/render/`](./paas/render) | `render.yaml` Blueprint with `/api/health/ready` health check and persistent disk |
| **Railway** | [`paas/railway/`](./paas/railway) | `railway.json` with `/data` persistent volume |

## What happens on first run

1. The container generates its database password, JWT signing key, NextAuth session secret, and
   Grafana password into `/data/postgres/secrets.env`.
2. You open `http://localhost:47100` and complete the 3-step **Setup Wizard**.
3. You create the admin account — this is the only account, and claims the instance permanently.
4. You connect Google Drive or S3-compatible storage using your own credentials.
5. Everything else has a working default and can be tuned live under **Settings**.

## Services & Ports

| Service | Port | Default Host Bind | What it is |
|---------|------|-------------------|------------|
| App | 47100 | `0.0.0.0:47100` | Web interface (proxies `/proxy/api/*` same-origin) |
| API | 47200 | `127.0.0.1:47200` | REST API, `/metrics`, and `/hangfire` dashboard |
| Grafana | 47500 | `127.0.0.1:47500` | Pre-provisioned dashboards (password in `/data/postgres/secrets.env`) |
| Prometheus | 47600 | `127.0.0.1:47600` | Metrics scraper |

Only port **47100** needs to be reachable for normal use. Internal admin ports bind to `127.0.0.1`
by default so they are never accidentally exposed on a public VPS.

## Everyday commands

```bash
docker logs -f torrentdrive
docker restart torrentdrive
```

To upgrade, re-run the install command or `docker compose pull && docker compose up -d`. It replaces
the container while preserving your database, downloads, and secrets across the three named volumes
(`torrentdrive-pgdata`, `torrentdrive-redis`, `torrentdrive-downloads`).

## Documentation

Full documentation lives at **[tc.gitnasr.com/docs](https://tc.gitnasr.com/docs)**.

## Repositories

| Repository | Contents |
|------------|----------|
| [backend](https://github.com/TorrentDrive/backend) | .NET 9 API, Hangfire orchestrator, hardened MonoTorrent engine, and upload workers |
| [frontend](https://github.com/TorrentDrive/frontend) | Next.js App Router web application |
| [website](https://github.com/TorrentDrive/website) | Landing site and Fumadocs documentation |
| [deploy](https://github.com/TorrentDrive/deploy) | Unified Dockerfile, Compose files, PaaS blueprints, installer, and CI |

## License

MIT — see [LICENSE](https://github.com/TorrentDrive/deploy/blob/main/LICENSE).