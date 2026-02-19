# 04-OpenClaw — Installatie

OpenClaw draait als Docker container. We clonen de broncode, bouwen een lokale image, configureren de omgeving en starten het geheel op met Docker Compose. De gateway is de kern van het systeem: hij verwerkt Telegram-berichten en AI-interacties.

---

## 1. Vereisten

- Docker geïnstalleerd (zie sectie 2 als dat nog niet het geval is)
- Git beschikbaar op de VPS
- Telegram Bot Token — aan te maken via @BotFather in Telegram
- VPS aangemaakt en firewall ingesteld (zie `01-Vps.md` en `02-Firewall.md`)

---

## 2. Docker installeren (als nog niet aanwezig)

```bash
ssh root@<IP> << 'EOF'
# Controleer of Docker al aanwezig is
docker --version 2>/dev/null && echo "Docker aanwezig" || {
  apt-get update
  apt-get install -y docker.io docker-compose-plugin
  systemctl enable --now docker
}
docker compose version
EOF
```

---

## 3. OpenClaw repo clonen en image bouwen

```bash
ssh root@<IP> << 'EOF'
cd /root
git clone https://github.com/openclaw/openclaw.git openclaw-src
cd openclaw-src

# Image bouwen (kan 5-10 minuten duren)
docker build -t openclaw:local .
echo "Build klaar: $(docker images openclaw:local --format '{{.Size}}')"
EOF
```

We bouwen een lokale image in plaats van een image te pullen uit een registry. Zo weet je precies wat er op je server draait — geen afhankelijkheid van externe registries, geen onverwachte updates.

---

## 4. Compose directory inrichten

```bash
ssh root@<IP> << 'EOF'
mkdir -p /root/openclaw
mkdir -p /root/clawd
mkdir -p /root/.openclaw
EOF
```

- `/root/openclaw/` — Docker Compose bestanden en `.env`
- `/root/clawd/` — werkruimte voor de gateway (bestanden, exports)
- `/root/.openclaw/` — persistente configuratie en geheugen van OpenClaw

---

## 5. docker-compose.yml aanmaken

```bash
ssh root@<IP> "cat > /root/openclaw/docker-compose.yml" << 'COMPOSE'
services:
  openclaw-gateway:
    image: ${OPENCLAW_IMAGE:-openclaw:local}
    container_name: openclaw-gateway
    dns: [8.8.8.8, 1.1.1.1]
    environment:
      OPENCLAW_GATEWAY_TOKEN: ${OPENCLAW_GATEWAY_TOKEN}
      TELEGRAM_BOT_TOKEN: ${TELEGRAM_BOT_TOKEN}
    volumes:
      - /root/.openclaw:/home/node/.openclaw
      - /root/clawd:/home/node/.openclaw/workspace
    networks:
      - openclaw-net
    restart: unless-stopped
    deploy:
      resources:
        limits:
          memory: 3g
          cpus: "2.0"
    command:
      - /bin/sh
      - -c
      - >-
        find /home/node/.openclaw -name "*.lock" -type f -delete 2>/dev/null || true;
        exec node dist/index.js gateway --allow-unconfigured
        --bind lan --port 18789

  caddy:
    image: caddy:2-alpine
    container_name: openclaw-caddy
    ports:
      - "127.0.0.1:80:80"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile:ro
      - /root/clawd/public:/srv/web:ro
      - caddy_data:/data
      - caddy_config:/config
    networks:
      - openclaw-net
    restart: unless-stopped
    depends_on:
      - openclaw-gateway
    deploy:
      resources:
        limits:
          memory: 256m
          cpus: "0.5"

networks:
  openclaw-net:
    driver: bridge

volumes:
  caddy_data:
  caddy_config:
COMPOSE
```

**Let op — Combinatie A of B (geen Caddy):** Als je Caddy niet installeert, verwijder de Caddy-service dan nu uit dit bestand — vóór je verdergaat met stap 6. Zonder Caddyfile crasht de Caddy-container bij elke start.

Verwijder uit `/root/openclaw/docker-compose.yml`:
- De volledige `caddy:` service-sectie (inclusief `ports`, `volumes`, `networks`, `restart`, etc.)
- De volumes `caddy_data:` en `caddy_config:` onderaan het bestand

Controleer daarna:
```bash
ssh root@<IP> "grep -c 'caddy' /root/openclaw/docker-compose.yml"
# Verwacht: 0
```

---

## 6. .env aanmaken

> **Pause:** Zorg dat je Telegram Bot Token bij de hand hebt voordat je verdergaat.
> Je haalt het op via @BotFather in Telegram. Het ziet eruit als: `123456789:ABCdef...`

```bash
# Stap 1: Sla je Telegram Bot Token op in een variabele
# Vervang de waarde hieronder met jouw token van @BotFather:
TELEGRAM_TOKEN="<jouw-telegram-bot-token>"

# Stap 2: Genereer een sterk gateway token (256-bits willekeurig)
GATEWAY_TOKEN=$(openssl rand -hex 32)
echo "Gateway token: $GATEWAY_TOKEN"
echo "Sla dit nu op in je password manager voordat je verdergaat."

# Stap 3: Schrijf de .env naar de VPS
ssh root@<IP> "cat > /root/openclaw/.env" << EOF
OPENCLAW_IMAGE=openclaw:local
OPENCLAW_GATEWAY_TOKEN=$GATEWAY_TOKEN
TELEGRAM_BOT_TOKEN=$TELEGRAM_TOKEN
EOF

# Beveiligen: alleen root mag dit lezen
ssh root@<IP> "chmod 600 /root/openclaw/.env"

# Verifieer (zonder geheimen te tonen)
ssh root@<IP> "stat -c '%a %n' /root/openclaw/.env && wc -l /root/openclaw/.env"
# Verwacht: 600 /root/openclaw/.env   en   3 (drie regels)
```

Het gateway token authenticeert alle API-aanroepen naar OpenClaw. Behandel het als een wachtwoord: deel het niet, sla het veilig op (bijvoorbeeld in een password manager), en roteer het als je denkt dat het gecompromitteerd is.

---

## 7. Rollback-tag aanmaken

```bash
ssh root@<IP> "docker tag openclaw:local openclaw:v$(date +%Y.%-m.%-d)"
ssh root@<IP> "docker images | grep openclaw"
```

Door de huidige werkende image te taggen met een datum, kun je altijd terugkeren naar deze versie als een toekomstige update problemen geeft. Doe dit ook vóór elke update. Rollback: `docker tag openclaw:v<datum> openclaw:local` en daarna `docker compose up -d`.

---

## 8. OpenClaw starten

```bash
ssh root@<IP> "cd /root/openclaw && docker compose up -d"
ssh root@<IP> "docker compose -f /root/openclaw/docker-compose.yml logs --tail=30 openclaw-gateway"
```

Controleer in de logs of de gateway succesvol opgestart is en verbinding heeft gemaakt met Telegram. Fouten met "invalid token" wijzen op een verkeerd Telegram Bot Token in de `.env`.

---

## 9. Eerste configuratie via de Control UI

Na het starten is OpenClaw bereikbaar via de Control UI (via Tailscale-URL of publiek adres, afhankelijk van je keuze). Stel via de UI het volgende in:

- **Anthropic API-sleutel** — vereist voor AI-functionaliteit. Zonder dit sleutel start de gateway wel, maar reageert de bot niet op AI-vragen.
- **Bot-configuratie** — koppel en configureer je Telegram-bots.
- Overige instellingen worden opgeslagen in `/root/.openclaw/openclaw.json`.

Als de bot start maar niet reageert op vragen, check dan de logs op ontbrekende API-sleutel:

```bash
ssh root@<IP> "docker logs openclaw-gateway --tail=20 | grep -i 'api\|anthropic\|key\|error'"
```

---

## 10. Vervolgstap

- Als Caddy gewenst is: zie `05-Caddy.md`
- Verificatie van de volledige installatie: zie `06-Verify.md`
