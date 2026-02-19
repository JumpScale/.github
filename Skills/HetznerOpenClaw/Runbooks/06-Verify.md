# 06-Verify — Verificatie & Checklist

Dit runbook verifieert of de installatie volledig en correct is uitgevoerd. Voer dit uit nadat alle gekozen runbooks zijn doorlopen. De checklist past zich aan op de gemaakte combinatiekeuze.

---

## 1. Quick check — alles draait?

```bash
# Containers
ssh root@<IP> "docker ps --format 'table {{.Names}}\t{{.Status}}'"
# Verwacht: openclaw-gateway Up (+ openclaw-caddy als Caddy geïnstalleerd)

# Tailscale (als gekozen)
ssh root@<IP> "tailscale status --peers=false"
# Verwacht: Tailscale IP zichtbaar, status: running
```

---

## 2. Security checks

```bash
# Hetzner Cloud Firewall actief?
hcloud firewall list

# UFW actief?
ssh root@<IP> "ufw status | head -3"
# Verwacht: Status: active

# SSH key-only (geen wachtwoordlogin)?
ssh root@<IP> "sshd -T | grep passwordauthentication"
# Verwacht: passwordauthentication no

# .env correct beveiligd?
ssh root@<IP> "stat -c '%a %n' /root/openclaw/.env"
# Verwacht: 600 /root/openclaw/.env

# Geen publieke poorten open behalve toegestane?
ssh root@<IP> "ss -tulnp | grep -E ':80|:443|:18789' | grep -v '127.0.0.1'"
# Verwacht: GEEN output (alles gebonden aan localhost of intern Docker netwerk)
```

---

## 3. Bereikbaarheidstest

```bash
# Poort 80 publiek NIET bereikbaar (geen Tailscale bypass)
curl -s --connect-timeout 5 http://<IP>/ && echo "PROBLEEM: poort 80 publiek bereikbaar!" || echo "OK: poort 80 geblokkeerd"

# Via Tailscale bereikbaar (als Tailscale gekozen)
curl -s -o /dev/null -w "%{http_code}" https://<TAILSCALE-HOSTNAME>/
# Verwacht: 200 of 401
```

---

## 4. Telegram-bot test

Stuur een bericht naar je bot in Telegram (bijv. `/start` of een gewone tekst). De bot moet reageren binnen 5-10 seconden.

Als de bot niet reageert:

```bash
ssh root@<IP> "docker logs openclaw-gateway --tail=30"
# Zoek naar: errors, "invalid token", connection refused
```

---

## 5. Volledige installatiechecklist

Vink af op basis van de gemaakte keuzes:

**Altijd:**
- [ ] VPS aangemaakt en bereikbaar via SSH
- [ ] Systeem volledig bijgewerkt (`apt-get upgrade`)
- [ ] SSH wachtwoordlogin uitgeschakeld
- [ ] Unattended-upgrades actief
- [ ] Hetzner Cloud Firewall actief (TCP 22, UDP 41641, ICMP)
- [ ] UFW actief (default deny, SSH limit)

**Als OpenClaw gekozen (B of C):**
- [ ] Docker-image gebouwd vanuit broncode
- [ ] docker-compose.yml aangemaakt
- [ ] `.env` mode 600
- [ ] Gateway token sterk (32+ bytes random)
- [ ] Telegram Bot Token correct ingevuld (niet de placeholder)
- [ ] Anthropic API-sleutel geconfigureerd via Control UI
- [ ] Containers draaien (`docker ps`)
- [ ] Bot reageert in Telegram
- [ ] Rollback-tag aangemaakt

**Als Tailscale gekozen:**
- [ ] Tailscale geïnstalleerd en verbonden
- [ ] VPS zichtbaar in Tailscale console
- [ ] OpenClaw bereikbaar via Tailscale URL
- [ ] UDP 41641 doorgelaten in Hetzner firewall

**Als Caddy gekozen (C):**
- [ ] Caddyfile aangemaakt
- [ ] Caddy container draait
- [ ] `/root/clawd/public/` map aangemaakt
- [ ] Webserver bereikbaar via `/web/`

**Als GEEN Tailscale:**
- [ ] HTTPS poort 443 open in Hetzner firewall
- [ ] Caddy geconfigureerd met domeinnaam + Let's Encrypt
- [ ] Fail2ban geïnstalleerd
- [ ] Gateway token genoteerd en veilig opgeslagen

---

## 6. Wat nu?

Na succesvolle verificatie:

1. **Noteer je inloggegevens** veilig op (IP, Tailscale-URL, gateway token)
2. **Stel backups in** via de Maintain-workflow
3. **Periodiek onderhoud**: voer eens per maand de Maintain-workflow uit
