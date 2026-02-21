# Stap 2: Twenty CRM installeren

## Doel
Twenty CRM draaiend in Docker met PostgreSQL en Redis.

---

## 2.1 Secrets genereren [Claude doet dit]

Genereer alle benodigde tokens en sla ze op in een `.env` bestand:

```bash
cd /opt/twenty-crm

cat > generate-secrets.sh << 'EOF'
#!/bin/bash
echo "ACCESS_TOKEN_SECRET=$(openssl rand -base64 32)"
echo "LOGIN_TOKEN_SECRET=$(openssl rand -base64 32)"
echo "REFRESH_TOKEN_SECRET=$(openssl rand -base64 32)"
echo "FILE_TOKEN_SECRET=$(openssl rand -base64 32)"
echo "POSTGRES_PASSWORD=$(openssl rand -base64 24)"
EOF
chmod +x generate-secrets.sh
./generate-secrets.sh > .env.secrets
```

**Toon de secrets NIET aan de gebruiker.** Schrijf ze direct naar het `.env` bestand.

## 2.2 Docker Compose bestand aanmaken [Claude doet dit]

```bash
cat > /opt/twenty-crm/docker-compose.yml << 'EOF'
name: twenty

services:
  server:
    image: twentycrm/twenty:latest
    ports:
      - "127.0.0.1:3000:3000"
    env_file: .env
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/healthz"]
      interval: 30s
      timeout: 10s
      retries: 5
    restart: always

  worker:
    image: twentycrm/twenty:latest
    command: ["yarn", "worker:prod"]
    env_file: .env
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    restart: always

  db:
    image: postgres:16
    volumes:
      - twenty-db:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: default
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: always

  redis:
    image: redis:8.2
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: always

volumes:
  twenty-db:
EOF
```

## 2.3 Environment bestand samenstellen [Claude doet dit]

Combineer secrets met configuratie:

```bash
cat > /opt/twenty-crm/.env << EOF
# Server
SERVER_URL=https://<DOMEIN>
FRONTEND_URL=https://<DOMEIN>

# Secrets (gegenereerd)
$(cat .env.secrets)

# Database
POSTGRES_ADMIN_URL=postgres://postgres:\${POSTGRES_PASSWORD}@db:5432/default

# Redis
REDIS_URL=redis://redis:6379

# Overig
SIGN_IN_PREFILLED=false
IS_SIGN_UP_DISABLED=false
EOF
rm .env.secrets
```

Vervang `<DOMEIN>` door het werkelijke domein van de gebruiker.

## 2.4 Opstarten [Claude doet dit]

```bash
cd /opt/twenty-crm && docker compose up -d
```

Wacht tot alle containers healthy zijn:
```bash
watch docker compose ps
```

## 2.5 Owner account aanmaken [Jij moet dit doen]

```
Twenty CRM draait. Ga naar https://<DOMEIN> in je browser.
Je ziet een registratiescherm -- maak hier je eerste account aan.
Dit wordt automatisch de admin/owner.

Kies een sterk wachtwoord en bewaar het veilig.
Geef een seintje als je klaar bent.
```

---

## Verificatie

```bash
docker compose ps                    # Alle containers "Up (healthy)"
curl -s -o /dev/null -w "%{http_code}" http://127.0.0.1:3000/healthz  # 200
```
