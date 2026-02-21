# 02 -- Trigger.dev v4 installeren

## Doel

Trigger.dev v4 draaien via Docker Compose met alle 9 vereiste services:
webapp, supervisor, docker-proxy, postgres, redis, electric, clickhouse, registry/ghcr.io, minio.

---

## Architectuur overzicht

v4 gebruikt twee losse Docker Compose stacks (gebaseerd op de officiele hosting/docker/ repo):

| Stack | Services | Compose bestand |
|-------|----------|-----------------|
| **webapp** | webapp, postgres, redis, electric, clickhouse, minio | `/opt/trigger-dev/webapp/docker-compose.yml` |
| **worker** | supervisor, docker-proxy, registry | `/opt/trigger-dev/worker/docker-compose.yml` |

Beide stacks lezen dezelfde `.env` file uit de parent directory.

---

## Stap 1: Directory structuur aanmaken [Claude doet dit]

```bash
ssh root@<SERVER_IP> << 'EOF'
mkdir -p /opt/trigger-dev/webapp
mkdir -p /opt/trigger-dev/worker
mkdir -p /opt/trigger-dev/clickhouse
EOF
```

## Stap 2: Environment bestand aanmaken [Jij moet dit doen]

```bash
ssh root@<SERVER_IP> << 'EOF'
# Secrets genereren
SESSION_SECRET=$(openssl rand -hex 16)
MAGIC_LINK_SECRET=$(openssl rand -hex 16)
ENCRYPTION_KEY=$(openssl rand -hex 16)
MANAGED_WORKER_SECRET=$(openssl rand -hex 16)
POSTGRES_PASSWORD=$(openssl rand -hex 24)
CLICKHOUSE_PASSWORD=$(openssl rand -hex 16)

cat > /opt/trigger-dev/.env << ENVEOF
# === Trigger.dev v4 Configuration ===

# Domain
TRIGGER_DOMAIN=<JOUW_DOMEIN>
APP_ORIGIN=https://<JOUW_DOMEIN>
LOGIN_ORIGIN=https://<JOUW_DOMEIN>
API_ORIGIN=https://<JOUW_DOMEIN>

# Image tags
TRIGGER_IMAGE_TAG=v4-beta

# Secrets
SESSION_SECRET=$SESSION_SECRET
MAGIC_LINK_SECRET=$MAGIC_LINK_SECRET
ENCRYPTION_KEY=$ENCRYPTION_KEY
MANAGED_WORKER_SECRET=$MANAGED_WORKER_SECRET

# PostgreSQL
POSTGRES_USER=postgres
POSTGRES_PASSWORD=$POSTGRES_PASSWORD
DATABASE_URL=postgresql://postgres:${POSTGRES_PASSWORD}@postgres:5432/trigger
DIRECT_URL=postgresql://postgres:${POSTGRES_PASSWORD}@postgres:5432/trigger

# Redis
REDIS_HOST=redis
REDIS_PORT=6379

# ClickHouse (NO ?secure=false in URL - crashes Node.js client!)
CLICKHOUSE_USER=default
CLICKHOUSE_PASSWORD=$CLICKHOUSE_PASSWORD
CLICKHOUSE_URL=http://default:${CLICKHOUSE_PASSWORD}@clickhouse:8123
SKIP_CLICKHOUSE_MIGRATIONS=1

# Electric
ELECTRIC_ORIGIN=http://electric:3000

# Supervisor
SUPERVISOR_HOST=supervisor
SUPERVISOR_PORT=8020

# Docker Proxy
DOCKER_PROXY_HOST=docker-proxy
DOCKER_PROXY_PORT=8021

# Docker Registry (ghcr.io aanbevolen boven local registry)
DOCKER_REGISTRY_URL=ghcr.io
DOCKER_REGISTRY_USERNAME=<GITHUB_USERNAME>
DOCKER_REGISTRY_NAMESPACE=<GITHUB_ORG_OF_USER>
DOCKER_REGISTRY_PASSWORD=<GITHUB_TOKEN_WRITE_PACKAGES>

# Network binding (alles op localhost)
PUBLISH_IP=127.0.0.1
POSTGRES_PUBLISH_IP=127.0.0.1
REDIS_PUBLISH_IP=127.0.0.1
CLICKHOUSE_PUBLISH_IP=127.0.0.1
MINIO_PUBLISH_IP=127.0.0.1
ELECTRIC_PUBLISH_IP=127.0.0.1

# Performance (aanpassen aan server resources)
NODE_MAX_OLD_SPACE_SIZE=1600

# Runtime
RUNTIME_PLATFORM=docker-compose
ENVEOF

chmod 600 /opt/trigger-dev/.env
echo "Environment bestand aangemaakt"
EOF
```

Vervang:
- `<JOUW_DOMEIN>` door je Trigger.dev (sub)domein, bijvoorbeeld `trigger.jouwbedrijf.nl`
- `<GITHUB_USERNAME>` door je GitHub gebruikersnaam
- `<GITHUB_ORG_OF_USER>` door je GitHub organisatie of gebruikersnaam
- `<GITHUB_TOKEN_WRITE_PACKAGES>` door een GitHub PAT met `write:packages` scope

### Waarom ghcr.io in plaats van local registry?

Local Docker registry geeft networking problemen met BuildKit in Docker Compose. ghcr.io werkt betrouwbaar. Je hebt een GitHub token nodig met `write:packages` scope.

## Stap 3: ClickHouse configuratie [Claude doet dit]

ClickHouse heeft memory tuning nodig, vooral op shared VPS:

```bash
ssh root@<SERVER_IP> "cat > /opt/trigger-dev/clickhouse/override.xml" << 'EOF'
<clickhouse>
    <profiles>
        <default>
            <max_memory_usage>1000000000</max_memory_usage>
            <max_memory_usage_for_all_queries>1500000000</max_memory_usage_for_all_queries>
        </default>
    </profiles>
</clickhouse>
EOF
```

## Stap 4: ClickHouse migraties draaien [Claude doet dit]

De `@clickhouse/client` Node.js library crasht op `?secure=false` in de URL. Maar Goose (de migratie-tool) heeft het nodig. Daarom draaien we migraties apart:

```bash
ssh root@<SERVER_IP> << 'EOF'
# Start alleen clickhouse voor migraties
cd /opt/trigger-dev/webapp
docker compose --env-file /opt/trigger-dev/.env up -d clickhouse

# Wacht tot clickhouse gezond is
sleep 10

# Draai migraties via een tijdelijke container met ?secure=false
# (de webapp entrypoint doet dit normaal, maar crasht op de URL parameter)
# Na migraties: SKIP_CLICKHOUSE_MIGRATIONS=1 in .env zorgt dat webapp ze overslaat
docker compose --env-file /opt/trigger-dev/.env run --rm \
  -e CLICKHOUSE_URL="http://default:<CLICKHOUSE_PW>@clickhouse:8123?secure=false" \
  webapp sh -c "cd /app && npx goose -dir ./internal-packages/clickhouse/migrations up"

echo "ClickHouse migraties voltooid"
EOF
```

**Let op:** Vervang `<CLICKHOUSE_PW>` met het wachtwoord uit de .env file. Na de eerste keer migraties draaien is dit niet meer nodig -- `SKIP_CLICKHOUSE_MIGRATIONS=1` zorgt dat de webapp ze overslaat bij elke volgende start.

## Stap 5: Webapp Compose bestand [Claude doet dit]

```bash
ssh root@<SERVER_IP> "cat > /opt/trigger-dev/webapp/docker-compose.yml" << 'EOF'
services:
  webapp:
    image: ghcr.io/triggerdotdev/trigger.dev:${TRIGGER_IMAGE_TAG:-v4-beta}
    container_name: trigger-webapp
    restart: unless-stopped
    ports:
      - "${PUBLISH_IP:-127.0.0.1}:8030:3000"
    environment:
      - DATABASE_URL=${DATABASE_URL}
      - DIRECT_URL=${DIRECT_URL}
      - REDIS_HOST=${REDIS_HOST}
      - REDIS_PORT=${REDIS_PORT}
      - SESSION_SECRET=${SESSION_SECRET}
      - MAGIC_LINK_SECRET=${MAGIC_LINK_SECRET}
      - ENCRYPTION_KEY=${ENCRYPTION_KEY}
      - MANAGED_WORKER_SECRET=${MANAGED_WORKER_SECRET}
      - LOGIN_ORIGIN=${LOGIN_ORIGIN}
      - APP_ORIGIN=${APP_ORIGIN}
      - API_ORIGIN=${API_ORIGIN}
      - ELECTRIC_ORIGIN=${ELECTRIC_ORIGIN}
      - CLICKHOUSE_URL=${CLICKHOUSE_URL}
      - SKIP_CLICKHOUSE_MIGRATIONS=${SKIP_CLICKHOUSE_MIGRATIONS:-0}
      - RUNTIME_PLATFORM=${RUNTIME_PLATFORM}
      - NODE_OPTIONS=--max-old-space-size=${NODE_MAX_OLD_SPACE_SIZE:-4096}
      - DOCKER_REGISTRY_URL=${DOCKER_REGISTRY_URL}
      - DOCKER_REGISTRY_USERNAME=${DOCKER_REGISTRY_USERNAME}
      - DOCKER_REGISTRY_PASSWORD=${DOCKER_REGISTRY_PASSWORD}
      - DOCKER_REGISTRY_NAMESPACE=${DOCKER_REGISTRY_NAMESPACE}
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
      clickhouse:
        condition: service_started
      electric:
        condition: service_started

  postgres:
    image: postgres:14
    container_name: trigger-postgres
    restart: unless-stopped
    environment:
      - POSTGRES_USER=${POSTGRES_USER:-postgres}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=trigger
    ports:
      - "${POSTGRES_PUBLISH_IP:-127.0.0.1}:5433:5432"
    volumes:
      - trigger_db:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER:-postgres}"]
      interval: 5s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7
    container_name: trigger-redis
    restart: unless-stopped
    ports:
      - "${REDIS_PUBLISH_IP:-127.0.0.1}:6389:6379"
    volumes:
      - trigger_redis:/data

  electric:
    image: electricsql/electric:1.2.4
    container_name: trigger-electric
    restart: unless-stopped
    environment:
      - DATABASE_URL=${DATABASE_URL}
    ports:
      - "${ELECTRIC_PUBLISH_IP:-127.0.0.1}:3060:3000"
    depends_on:
      postgres:
        condition: service_healthy

  clickhouse:
    image: bitnami/clickhouse:latest
    container_name: trigger-clickhouse
    restart: unless-stopped
    environment:
      - CLICKHOUSE_ADMIN_USER=${CLICKHOUSE_USER:-default}
      - CLICKHOUSE_ADMIN_PASSWORD=${CLICKHOUSE_PASSWORD}
    ports:
      - "${CLICKHOUSE_PUBLISH_IP:-127.0.0.1}:8123:8123"
    volumes:
      - trigger_clickhouse:/bitnami/clickhouse
      - /opt/trigger-dev/clickhouse/override.xml:/bitnami/clickhouse/etc/conf.d/override.xml:ro

  minio:
    image: bitnami/minio:latest
    container_name: trigger-minio
    restart: unless-stopped
    environment:
      - MINIO_ROOT_USER=minioadmin
      - MINIO_ROOT_PASSWORD=minioadmin
    ports:
      - "${MINIO_PUBLISH_IP:-127.0.0.1}:9000:9000"
      - "${MINIO_PUBLISH_IP:-127.0.0.1}:9001:9001"
    volumes:
      - trigger_minio:/bitnami/minio/data

volumes:
  trigger_db:
  trigger_redis:
  trigger_clickhouse:
  trigger_minio:
EOF
```

## Stap 6: Worker Compose bestand [Claude doet dit]

```bash
ssh root@<SERVER_IP> "cat > /opt/trigger-dev/worker/docker-compose.yml" << 'EOF'
services:
  supervisor:
    image: ghcr.io/triggerdotdev/supervisor:${TRIGGER_IMAGE_TAG:-v4-beta}
    container_name: trigger-supervisor
    restart: unless-stopped
    environment:
      - MANAGED_WORKER_SECRET=${MANAGED_WORKER_SECRET}
      - TRIGGER_API_URL=http://webapp:3000
      - DOCKER_PROXY_HOST=${DOCKER_PROXY_HOST:-docker-proxy}
      - DOCKER_PROXY_PORT=${DOCKER_PROXY_PORT:-8021}
      - REDIS_HOST=${REDIS_HOST}
      - REDIS_PORT=${REDIS_PORT}
    networks:
      - default
      - webapp_default

  docker-proxy:
    image: ghcr.io/triggerdotdev/docker-proxy:${TRIGGER_IMAGE_TAG:-v4-beta}
    container_name: trigger-docker-proxy
    restart: unless-stopped
    environment:
      - DOCKER_PROXY_PORT=${DOCKER_PROXY_PORT:-8021}
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - default
      - webapp_default

networks:
  webapp_default:
    external: true
    name: webapp_default
EOF
```

**Let op:** De worker-stack verbindt met de webapp-stack via een extern Docker network (`webapp_default`). Dit network wordt automatisch aangemaakt door de webapp compose stack.

## Stap 7: Management script aanmaken [Claude doet dit]

```bash
ssh root@<SERVER_IP> "cat > /opt/trigger-dev/trigger-ctl.sh" << 'SCRIPT'
#!/bin/bash
set -euo pipefail

ENV_FILE="/opt/trigger-dev/.env"
WEBAPP_COMPOSE="/opt/trigger-dev/webapp/docker-compose.yml"
WORKER_COMPOSE="/opt/trigger-dev/worker/docker-compose.yml"

case "${1:-help}" in
  up)
    docker compose -f "$WEBAPP_COMPOSE" --env-file "$ENV_FILE" up -d
    sleep 5
    docker compose -f "$WORKER_COMPOSE" --env-file "$ENV_FILE" up -d
    echo "Trigger.dev v4 gestart"
    ;;
  down)
    docker compose -f "$WORKER_COMPOSE" --env-file "$ENV_FILE" down
    docker compose -f "$WEBAPP_COMPOSE" --env-file "$ENV_FILE" down
    echo "Trigger.dev v4 gestopt"
    ;;
  restart)
    $0 down
    $0 up
    ;;
  logs)
    SERVICE="${2:-}"
    if [ -n "$SERVICE" ]; then
      docker compose -f "$WEBAPP_COMPOSE" --env-file "$ENV_FILE" logs -f --tail=50 "$SERVICE" 2>/dev/null || \
      docker compose -f "$WORKER_COMPOSE" --env-file "$ENV_FILE" logs -f --tail=50 "$SERVICE"
    else
      docker compose -f "$WEBAPP_COMPOSE" --env-file "$ENV_FILE" logs -f --tail=30 &
      docker compose -f "$WORKER_COMPOSE" --env-file "$ENV_FILE" logs -f --tail=30
    fi
    ;;
  status)
    echo "=== Webapp Stack ==="
    docker compose -f "$WEBAPP_COMPOSE" --env-file "$ENV_FILE" ps
    echo ""
    echo "=== Worker Stack ==="
    docker compose -f "$WORKER_COMPOSE" --env-file "$ENV_FILE" ps
    ;;
  *)
    echo "Usage: $0 {up|down|restart|logs [service]|status}"
    ;;
esac
SCRIPT

ssh root@<SERVER_IP> "chmod +x /opt/trigger-dev/trigger-ctl.sh"
```

## Stap 8: Docker login voor ghcr.io [Jij moet dit doen]

```bash
ssh root@<SERVER_IP> "docker login ghcr.io"
```

Voer je GitHub gebruikersnaam en een Personal Access Token in met `write:packages` scope.

## Stap 9: Trigger.dev starten [Claude doet dit]

```bash
ssh root@<SERVER_IP> "/opt/trigger-dev/trigger-ctl.sh up"
ssh root@<SERVER_IP> "/opt/trigger-dev/trigger-ctl.sh status"
```

Wacht tot alle containers draaien.

## Stap 10: Eerste login [Jij moet dit doen]

Trigger.dev gebruikt magic link authenticatie. De login-link verschijnt in de container logs:

```bash
ssh root@<SERVER_IP> "/opt/trigger-dev/trigger-ctl.sh logs webapp"
```

```
Ga naar: https://<jouw-domein>/ (na reverse proxy setup)
Of tijdelijk: http://<SERVER_IP>:8030 (als poort open staat)

Voer je e-mailadres in. De magic link verschijnt in de container logs.
```

## Stap 11: Deploy flow configureren [Jij moet dit doen]

Trigger.dev v4 task deployment moet vanaf de VPS zelf (niet vanaf je lokale Mac -- BuildKit networking issues):

```bash
# Op de VPS:
# 1. Installeer Trigger.dev CLI
npx trigger.dev@latest login --api-url https://<JOUW_DOMEIN>

# 2. Deploy vanuit je project directory
npx trigger.dev@latest deploy --api-url https://<JOUW_DOMEIN>
```

CLI config wordt opgeslagen in `/root/.config/trigger/config.json`.

## Verificatie

```bash
ssh root@<SERVER_IP> << 'EOF'
echo "=== Containers ==="
docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}' | grep trigger

echo ""
echo "=== Health ==="
curl -s http://localhost:8030/ | head -5
echo ""

echo ""
echo "=== Logs (laatste 10 regels) ==="
/opt/trigger-dev/trigger-ctl.sh logs webapp 2>&1 | tail -10
EOF
```

Verwacht:
- 9 containers draaien (webapp, postgres, redis, electric, clickhouse, minio, supervisor, docker-proxy, en evt. registry)
- Webapp bereikbaar op localhost:8030
