# 02 -- Trigger.dev installeren

## Doel

Trigger.dev draaien via Docker Compose met PostgreSQL, Redis, en alle vereiste services.

---

## Stap 1: Docker Compose bestand aanmaken

```bash
ssh root@<SERVER_IP> "cat > /opt/trigger/docker-compose.yml" << 'EOF'
services:
  webapp:
    image: ghcr.io/triggerdotdev/trigger.dev:v3
    container_name: trigger-webapp
    restart: unless-stopped
    ports:
      - "127.0.0.1:3040:3030"
    environment:
      - DATABASE_URL=postgresql://trigger:${DB_PASSWORD}@postgres:5432/trigger
      - DIRECT_URL=postgresql://trigger:${DB_PASSWORD}@postgres:5432/trigger
      - REDIS_HOST=redis
      - REDIS_PORT=6379
      - MAGIC_LINK_SECRET=${MAGIC_LINK_SECRET}
      - SESSION_SECRET=${SESSION_SECRET}
      - ENCRYPTION_KEY=${ENCRYPTION_KEY}
      - LOGIN_ORIGIN=https://${TRIGGER_DOMAIN}
      - APP_ORIGIN=https://${TRIGGER_DOMAIN}
      - V3_ENABLED=true
      - COORDINATOR_HOST=coordinator
      - COORDINATOR_PORT=8020
      - ELECTRIC_ORIGIN=http://electric:3000
      - DOCKER_PROVIDER_HOST=docker-provider
      - DOCKER_PROVIDER_PORT=8021
      - RUNTIME_PLATFORM=docker-compose
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started

  postgres:
    image: postgres:16-alpine
    container_name: trigger-postgres
    restart: unless-stopped
    environment:
      - POSTGRES_USER=trigger
      - POSTGRES_PASSWORD=${DB_PASSWORD}
      - POSTGRES_DB=trigger
    volumes:
      - trigger_db:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U trigger"]
      interval: 5s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    container_name: trigger-redis
    restart: unless-stopped
    volumes:
      - trigger_redis:/data

  coordinator:
    image: ghcr.io/triggerdotdev/coordinator:v3
    container_name: trigger-coordinator
    restart: unless-stopped
    environment:
      - COORDINATOR_PORT=8020
      - REDIS_HOST=redis
      - REDIS_PORT=6379
    depends_on:
      - redis

  docker-provider:
    image: ghcr.io/triggerdotdev/provider/docker:v3
    container_name: trigger-docker-provider
    restart: unless-stopped
    environment:
      - PROVIDER_PORT=8021
      - COORDINATOR_HOST=coordinator
      - COORDINATOR_PORT=8020
      - REDIS_HOST=redis
      - REDIS_PORT=6379
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    depends_on:
      - coordinator
      - redis

  electric:
    image: electricsql/electric:latest
    container_name: trigger-electric
    restart: unless-stopped
    environment:
      - DATABASE_URL=postgresql://trigger:${DB_PASSWORD}@postgres:5432/trigger
    depends_on:
      postgres:
        condition: service_healthy

volumes:
  trigger_db:
  trigger_redis:
EOF
```

## Stap 2: Environment bestand aanmaken [Jij moet dit doen]

```bash
ssh root@<SERVER_IP> << 'EOF'
# Secrets genereren
DB_PASSWORD=$(openssl rand -hex 16)
MAGIC_LINK_SECRET=$(openssl rand -hex 32)
SESSION_SECRET=$(openssl rand -hex 32)
ENCRYPTION_KEY=$(openssl rand -hex 16)

cat > /opt/trigger/.env << ENVEOF
DB_PASSWORD=$DB_PASSWORD
MAGIC_LINK_SECRET=$MAGIC_LINK_SECRET
SESSION_SECRET=$SESSION_SECRET
ENCRYPTION_KEY=$ENCRYPTION_KEY
TRIGGER_DOMAIN=<JOUW_DOMEIN>
ENVEOF

chmod 600 /opt/trigger/.env
echo "Environment bestand aangemaakt"
EOF
```

Vervang `<JOUW_DOMEIN>` door je Trigger.dev (sub)domein, bijvoorbeeld `trigger.jouwbedrijf.nl`.

## Stap 3: Trigger.dev starten

```bash
ssh root@<SERVER_IP> << 'EOF'
cd /opt/trigger
docker compose up -d
docker compose logs -f --tail=30 webapp
EOF
```

Wacht tot je ziet dat de webapp gestart is.

## Stap 4: Eerste login [Jij moet dit doen]

Trigger.dev gebruikt magic link authenticatie. De login-link verschijnt in de container logs:

```bash
ssh root@<SERVER_IP> << 'EOF'
cd /opt/trigger
# Ga naar het dashboard en voer je e-mailadres in
# De magic link verschijnt in de logs:
docker compose logs webapp | grep -i "magic\|login\|email"
EOF
```

```
Ga naar: https://<jouw-domein>/ (na reverse proxy setup)
Of tijdelijk: http://<SERVER_IP>:3040 (als poort open staat)

Voer je e-mailadres in. De magic link verschijnt in de container logs.
```

## Verificatie

```bash
ssh root@<SERVER_IP> << 'EOF'
echo "=== Containers ==="
docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}' | grep trigger

echo "=== Health ==="
curl -s http://localhost:3040/ | head -5
echo ""

echo "=== Logs (laatste 10 regels) ==="
docker compose -f /opt/trigger/docker-compose.yml logs --tail=10 webapp
EOF
```

Verwacht:
- Alle 6 containers draaien (webapp, postgres, redis, coordinator, docker-provider, electric)
- Webapp bereikbaar op localhost:3040
