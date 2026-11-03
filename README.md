# Local dev infrastructure

Docker Compose stack for local development: databases, cache, search, and optional AI and monitoring services. Services share the `local-infra-network` bridge network and use named volumes for data persistence.

## Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) or Docker Engine with the Compose plugin
- Enough disk and RAM for the services you enable (OpenSearch is the heaviest core service)

## Quick start

1. Clone this repository and open a shell in the project root.

2. Create a local env file (optional but recommended for non-default credentials):

   ```bash
   cp .env.example .env
   ```

3. Start the **core** stack (MongoDB, OpenSearch, Redis, DocumentDB):

   ```bash
   docker compose up -d
   ```

4. Check status:

   ```bash
   docker compose ps
   ```

Core services expose health checks; wait until they report `healthy` before connecting from apps.

## Environment variables

Compose reads a `.env` file in this directory automatically and substitutes values in `docker-compose.yml` as `${VAR:-default}`.

| Variable | Used by | Default |
|----------|---------|---------|
| `MONGO_USERNAME` | MongoDB | `admin` |
| `MONGO_PASSWORD` | MongoDB | `password` |
| `DOCDB_USERNAME` | DocumentDB | `admin` |
| `DOCDB_PASSWORD` | DocumentDB | `password` |
| `POSTGRES_USER` | PostgreSQL (profile) | `admin` |
| `POSTGRES_PASSWORD` | PostgreSQL (profile) | `password` |
| `POSTGRES_DB` | PostgreSQL (profile) | `senserank` |
| `GRAFANA_USER` | Grafana (profile) | `admin` |
| `GRAFANA_PASSWORD` | Grafana (profile) | `admin` |

Copy `.env.example` to `.env` and edit as needed. `.env` is gitignored.

Other ways to pass values:

```bash
# Alternate env file
docker compose --env-file .env.local up -d

# One-off override
MONGO_PASSWORD=secret docker compose up -d

# Inspect resolved config (may show secrets)
docker compose config
```

## Services and ports

### Core (always started)

| Service | Host port | Container name |
|---------|-----------|----------------|
| MongoDB | 27017 | senserank-mongodb |
| OpenSearch | 9200 | senserank-opensearch |
| Redis | 6379 | senserank-redis |
| DocumentDB (local) | 10260 | senserank-docdb |

DocumentDB uses TLS locally; clients typically need `--tlsAllowInvalidCertificates` (or equivalent) for dev.

### Optional profiles

Enable extra services with `--profile`:

| Profile | Services | Host ports |
|---------|----------|------------|
| `ai` | Ollama | 11434 |
| `database` | PostgreSQL (pgvector) | 5432 |
| `monitoring` | Prometheus, Grafana | 9090, 3000 |

Examples:

```bash
# Core + PostgreSQL
docker compose --profile database up -d

# Core + Ollama
docker compose --profile ai up -d

# Core + monitoring
docker compose --profile monitoring up -d

# Everything
docker compose --profile ai --profile database --profile monitoring up -d
```

The **monitoring** profile mounts `./prometheus/prometheus.yml` into Prometheus. Ensure that file exists before starting the profile.

## Common commands

```bash
# Stop and remove containers (volumes kept)
docker compose down

# Stop and remove containers and named volumes (data loss)
docker compose down -v

# Follow logs for one service
docker compose logs -f mongodb

# Restart a single service
docker compose restart redis
```

## Connecting from other projects

- **Network:** `local-infra-network` (external name is fixed in compose)
- **From another Compose project:** attach your app service to that network, or use `host.docker.internal` / `localhost` with the host ports above when the app runs on the host.

Example connection strings (defaults):

- MongoDB: `mongodb://admin:password@localhost:27017`
- Redis: `redis://localhost:6379`
- OpenSearch: `http://localhost:9200` (security plugin disabled in this stack)
- PostgreSQL: `postgresql://admin:password@localhost:5432/senserank`

Adjust usernames, passwords, and database names to match your `.env`.

## License

MIT — see [LICENSE](LICENSE).
