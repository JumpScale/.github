# 02-OpenClaw — Installatie en Configuratie

OpenClaw draait als Docker container. We clonen de broncode, bouwen een lokale image,
configureren de omgeving en starten het geheel op met Docker Compose.

---

## 1. Vereisten

- Docker geïnstalleerd (zie [01-Docker.md](01-Docker.md))
- Git beschikbaar op de VPS
- Telegram Bot Token — aan te maken via @BotFather in Telegram
- VPS met SSH-toegang

---

## 2. OpenClaw repo clonen en image bouwen

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

We bouwen een lokale image in plaats van een image te pullen uit een registry. Zo weet je
precies wat er op je server draait.

---

## 3. Compose directory inrichten

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

## 4. docker-compose.yml aanmaken

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

networks:
  openclaw-net:
    driver: bridge
COMPOSE
```

> **Caddy nodig?** Als je ook Caddy wilt installeren (voor het delen van bestanden),
> laat dit bestand staan — de Caddy-service wordt toegevoegd in [03-Caddy.md](03-Caddy.md).

---

## 5. .env aanmaken

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

---

## 6. Rollback-tag aanmaken

```bash
ssh root@<IP> "docker tag openclaw:local openclaw:v$(date +%Y.%-m.%-d)"
ssh root@<IP> "docker images | grep openclaw"
```

Door de huidige werkende image te taggen met een datum, kun je altijd terugkeren naar
deze versie als een toekomstige update problemen geeft.

---

## 7. OpenClaw starten

```bash
ssh root@<IP> "cd /root/openclaw && docker compose up -d"
ssh root@<IP> "docker compose -f /root/openclaw/docker-compose.yml logs --tail=30 openclaw-gateway"
```

Controleer in de logs of de gateway succesvol opgestart is en verbinding heeft gemaakt
met Telegram.

---

## 8. Eerste configuratie via de Control UI

Na het starten is OpenClaw bereikbaar via de Control UI (via Tailscale-URL of publiek adres,
afhankelijk van je server-setup). Stel via de UI het volgende in:

- **Anthropic API-sleutel** — vereist voor AI-functionaliteit
- **Bot-configuratie** — koppel en configureer je Telegram-bots
- Overige instellingen worden opgeslagen in `/root/.openclaw/openclaw.json`

Als de bot start maar niet reageert:

```bash
ssh root@<IP> "docker logs openclaw-gateway --tail=20 | grep -i 'api\|anthropic\|key\|error'"
```

---

## Vervolgstap

- Als Caddy gewenst is: zie [03-Caddy.md](03-Caddy.md)
- Verificatie: zie [04-Verify.md](04-Verify.md)
