# 04-Verify — OpenClaw Verificatie-Checklist

Dit runbook verifieert of de OpenClaw-installatie volledig en correct werkt.

---

## 1. Quick check — alles draait?

```bash
# Containers
ssh root@<IP> "docker ps --format 'table {{.Names}}\t{{.Status}}'"
# Verwacht: openclaw-gateway Up (+ openclaw-caddy als Caddy geïnstalleerd)
```

---

## 2. Security checks

```bash
# .env correct beveiligd?
ssh root@<IP> "stat -c '%a %n' /root/openclaw/.env"
# Verwacht: 600 /root/openclaw/.env

# Gateway niet publiek bereikbaar?
ssh root@<IP> "ss -tulnp | grep ':18789' | grep -v '172\.\|127\.'"
# Verwacht: GEEN output (gateway alleen via intern Docker-netwerk)
```

---

## 3. Telegram-bot test

Stuur een bericht naar je bot in Telegram (bijv. `/start` of een gewone tekst).
De bot moet reageren binnen 5-10 seconden.

Als de bot niet reageert:

```bash
ssh root@<IP> "docker logs openclaw-gateway --tail=30"
# Zoek naar: errors, "invalid token", connection refused
```

---

## 4. Caddy test (als geïnstalleerd)

```bash
# Container draait?
ssh root@<IP> "docker ps | grep caddy"

# Bereikbaar?
ssh root@<IP> "curl -s -o /dev/null -w '%{http_code}' http://localhost:80/"
# Verwacht: 200 of 502 (gateway nog aan het opstarten)

# Test bestand delen:
ssh root@<IP> "echo 'test' > /root/clawd/public/test.txt"
ssh root@<IP> "curl -s http://localhost:80/web/test.txt"
# Verwacht: "test"
ssh root@<IP> "rm /root/clawd/public/test.txt"
```

---

## 5. Installatiechecklist

**Altijd:**
- [ ] Docker-image gebouwd vanuit broncode
- [ ] docker-compose.yml aangemaakt
- [ ] `.env` mode 600
- [ ] Gateway token sterk (32+ bytes random)
- [ ] Telegram Bot Token correct ingevuld
- [ ] Anthropic API-sleutel geconfigureerd via Control UI
- [ ] Containers draaien (`docker ps`)
- [ ] Bot reageert in Telegram
- [ ] Rollback-tag aangemaakt

**Als Caddy gekozen:**
- [ ] Caddyfile aangemaakt
- [ ] Caddy container draait
- [ ] `/root/clawd/public/` map aangemaakt
- [ ] Webserver bereikbaar via `/web/`

---

## 6. Wat nu?

Na succesvolle verificatie:

1. **Noteer je gegevens veilig** (gateway token, Telegram bot naam)
2. **Stel backups in** via de Maintain-workflow
3. **Periodiek onderhoud**: voer eens per maand de Maintain-workflow uit
