# Archivematica Docker Compose

Production-ready Docker Compose configuration for deploying [Archivematica](https://www.archivematica.org/).

## Overview

This repository provides a self-contained Docker Compose setup for Archivematica that:

- Exposes Dashboard on port 8000
- Exposes Storage Service on port 8001
- Includes all required services (MySQL, Elasticsearch, ClamAV, etc.)
- Uses production-safe defaults
- No nginx included (bring your own reverse proxy for SSL)

## Prerequisites

- Docker Engine ≥ 23.0
- Docker Compose ≥ 2.17
- At least 8GB RAM (16GB recommended)
- `vm.max_map_count=262144` set on host (required for Elasticsearch)

To set the Elasticsearch requirement:

```bash
sudo sysctl -w vm.max_map_count=262144
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
```

## Deployment

### 1. Clone This Repository

```bash
git clone --recurse-submodules https://github.com/yourusername/archivematica-docker.git
cd archivematica-docker
```

Or if already cloned:

```bash
git submodule update --init --recursive
```

### 2. Generate Secure Credentials

```bash
# Django secret keys (generate two separate keys)
python3 -c "import secrets; print(secrets.token_urlsafe(50))"
python3 -c "import secrets; print(secrets.token_urlsafe(50))"

# MySQL passwords (generate two separate passwords)
openssl rand -base64 32 | tr -d '=+/'
openssl rand -base64 32 | tr -d '=+/'
```

### 3. Configure Environment

Copy the example environment file and add your generated secrets:

```bash
cp env.example .env
```

Edit `.env` and replace all `REPLACE_WITH_*` placeholders with your generated values.

### 4. Deploy

```bash
docker compose up -d --build
```

### 5. Initialize Databases

Wait for services to be healthy, then run migrations:

```bash
docker compose exec archivematica-dashboard /src/dashboard/src/manage.py migrate
docker compose exec archivematica-storage-service /src/storage_service/manage.py migrate
```

Create admin users:

```bash
docker compose exec archivematica-dashboard /src/dashboard/src/manage.py createsuperuser
docker compose exec archivematica-storage-service /src/storage_service/manage.py createsuperuser
```

### 6. Access

- **Dashboard**: http://your-server:8000
- **Storage Service**: http://your-server:8001

For production, place a reverse proxy (nginx, Caddy, Traefik) in front to handle SSL.

## Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `DJANGO_SECRET_KEY` | Dashboard secret key | (generated) |
| `SS_DJANGO_SECRET_KEY` | Storage Service secret key | (generated) |
| `MYSQL_ROOT_PASSWORD` | MySQL root password | (generated) |
| `MYSQL_USER` | MySQL user | `archivematica` |
| `MYSQL_PASSWORD` | MySQL user password | (generated) |
| `DJANGO_ALLOWED_HOSTS` | Allowed hostnames | `*` or `yourdomain.com` |
| `ES_HEAP_SIZE` | Elasticsearch memory | `1g` |
| `AM_SEARCH_ENABLED` | Enable search | `true` |
| `CLAMAV_MAX_FILE_SIZE` | Max file size for scanning | `2000M` |
| `CLAMAV_MAX_SCAN_SIZE` | Max scan size | `2000M` |
| `CLAMAV_MAX_STREAM_LENGTH` | Max stream length | `2000M` |
| `AM_GUNICORN_WORKERS` | Dashboard workers | `4` |
| `SS_GUNICORN_WORKERS` | Storage Service workers | `4` |
| `AM_GUNICORN_TIMEOUT` | Request timeout (seconds) | `172800` |
| `SS_GUNICORN_TIMEOUT` | Request timeout (seconds) | `172800` |
| `AM_CAPTURE_CLIENT_SCRIPT_OUTPUT` | Capture script output | `true` |

See `env.example` for a complete template.

## Services

| Service | Port | Description |
|---------|------|-------------|
| archivematica-dashboard | 8000 | Main web interface |
| archivematica-storage-service | 8001 | Storage management interface |
| mysql | 3306 | Database (internal) |
| elasticsearch | 9200 | Search engine (internal) |
| gearmand | 4730 | Job queue (internal) |
| redis | 6379 | Cache (internal) |
| clamavd | 3310 | Antivirus (internal) |
| fits | 2113 | File identification (internal) |

## Memory Requirements

| Service | Approximate RAM |
|---------|-----------------|
| Elasticsearch | 1GB+ |
| ClamAV | 500MB |
| MySQL | 500MB |
| Dashboard | 150MB |
| Storage Service | 100MB |
| MCP Client | 50MB |
| MCP Server | 50MB |
| **Total** | **~2.5GB minimum** |

Recommend 8GB+ RAM for production workloads.

## Useful Commands

```bash
# View logs
docker compose logs -f

# View specific service logs
docker compose logs -f archivematica-dashboard

# Check service status
docker compose ps

# Restart a service
docker compose restart archivematica-dashboard

# Stop everything
docker compose down

# Stop and remove volumes (WARNING: destroys data)
docker compose down -v

# Rebuild after changes
docker compose up -d --build
```

## Troubleshooting

### Elasticsearch fails to start

Ensure `vm.max_map_count` is set on the host:

```bash
sudo sysctl vm.max_map_count=262144
```

### Services restarting continuously

Check logs:

```bash
docker compose logs -f archivematica-dashboard
docker compose logs -f mysql
```

### Database connection errors

Wait for MySQL health check to pass before running migrations:

```bash
docker compose ps
```

## Adding Submodules (for repository setup)

If setting up this repository from scratch:

```bash
git submodule add -b qa/1.x https://github.com/artefactual/archivematica.git submodules/archivematica
git submodule add -b qa/0.x https://github.com/artefactual/archivematica-storage-service.git submodules/archivematica-storage-service
```

## License

This deployment configuration is provided as-is. Archivematica is licensed under the [AGPL-3.0](https://www.archivematica.org/en/docs/archivematica-1.18/user-manual/overview/intro/#license).

## Links

- [Archivematica Documentation](https://www.archivematica.org/en/docs/)
- [Archivematica GitHub](https://github.com/artefactual/archivematica)