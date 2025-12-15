# Archivematica Docker Compose

Production-ready Docker Compose configuration for deploying [Archivematica](https://www.archivematica.org/).

Based on the official development environment at [artefactual/archivematica/hack](https://github.com/artefactual/archivematica/tree/qa/1.x/hack).

## Overview

This setup:

- Exposes Dashboard on port 8000
- Exposes Storage Service on port 8001
- Includes all required services (MySQL, Elasticsearch, ClamAV, Gearman)
- Uses production-safe defaults
- No nginx included (bring your own reverse proxy for SSL)

## Prerequisites

- Docker Engine ≥ 23.0
- Docker Compose ≥ 2.17
- At least 8GB RAM (16GB recommended)
- `vm.max_map_count=262144` set on host (required for Elasticsearch)

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
# Django secret key
python3 -c "import secrets; print(secrets.token_urlsafe(50))"

# MySQL passwords (generate two)
openssl rand -base64 32 | tr -d '=+/'
openssl rand -base64 32 | tr -d '=+/'
```

### 3. Configure Environment

```bash
cp env.example .env
```

Edit `.env` and replace all `REPLACE_WITH_*` placeholders.

### 4. Deploy

```bash
docker compose up -d --build
```

### 5. Initialize Databases

Wait for services to be healthy:

```bash
docker compose ps
```

Then run migrations:

```bash
docker compose exec archivematica-dashboard /src/hack/manage.py migrate --settings=archivematica.dashboard.settings.local
docker compose exec archivematica-storage-service /src/manage.py migrate
```

Create admin users:

```bash
docker compose exec archivematica-dashboard /src/hack/manage.py createsuperuser --settings=archivematica.dashboard.settings.local
docker compose exec archivematica-storage-service /src/manage.py createsuperuser
```

### 6. Access

- **Dashboard**: http://your-server:8000
- **Storage Service**: http://your-server:8001

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `DJANGO_SECRET_KEY` | Dashboard secret key | (required) |
| `MYSQL_ROOT_PASSWORD` | MySQL root password | (required) |
| `MYSQL_USER` | MySQL user | `archivematica` |
| `MYSQL_PASSWORD` | MySQL user password | (required) |
| `ES_HEAP_SIZE` | Elasticsearch memory | `512m` |
| `AM_SEARCH_ENABLED` | Enable search | `true` |
| `CLAMAV_MAX_FILE_SIZE_MB` | Max file size (MB) | `42` |
| `CLAMAV_MAX_SCAN_SIZE_MB` | Max scan size (MB) | `42` |
| `CLAMAV_MAX_STREAM_LENGTH_MB` | Max stream (MB) | `100` |
| `AM_GUNICORN_WORKERS` | Dashboard workers | `4` |
| `SS_GUNICORN_WORKERS` | Storage workers | `4` |
| `AM_GUNICORN_TIMEOUT` | Timeout (seconds) | `172800` |
| `SS_GUNICORN_TIMEOUT` | Timeout (seconds) | `172800` |

## Services

| Service | Description |
|---------|-------------|
| archivematica-dashboard | Main web interface (port 8000) |
| archivematica-storage-service | Storage management (port 8001) |
| archivematica-mcp-server | Job coordination |
| archivematica-mcp-client | Job processing |
| mysql | Database (Percona 8.4) |
| elasticsearch | Search engine (ES 8.19) |
| gearmand | Job queue |
| clamavd | Antivirus scanning |

## Useful Commands

```bash
docker compose logs -f                    # View all logs
docker compose logs -f archivematica-dashboard  # Dashboard logs
docker compose ps                         # Service status
docker compose restart archivematica-dashboard  # Restart service
docker compose down                       # Stop everything
docker compose down -v                    # Stop and remove volumes
docker compose up -d --build              # Rebuild and start
```

## Adding Submodules

If setting up from scratch:

```bash
git submodule add -b qa/1.x https://github.com/artefactual/archivematica.git submodules/archivematica
git submodule add -b qa/0.x https://github.com/artefactual/archivematica-storage-service.git submodules/archivematica-storage-service
```

## License

Archivematica is licensed under [AGPL-3.0](https://www.archivematica.org/en/docs/archivematica-1.18/user-manual/overview/intro/#license).

## Links

- [Archivematica Documentation](https://www.archivematica.org/en/docs/)
- [Archivematica GitHub](https://github.com/artefactual/archivematica)