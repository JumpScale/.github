# Maintain Workflow

Periodiek onderhoud voor een bestaande OpenClaw-installatie op Hetzner.
Aanbevolen: eens per maand uitvoeren, of na een nieuwe OpenClaw-release.

---

## Fase 0: Verbinding opzetten

Vraag de gebruiker naar de servernaam of het IP-adres van de VPS:

```bash
# Ophalen via hcloud als je de servernaam weet:
IP=$(hcloud server ip <SERVERNAAM>)
echo "VPS IP: $IP"

# Of vraag de gebruiker het IP direct in te vullen:
# IP="<IP-adres>"
```

Gebruik `$IP` in alle verdere SSH-commando's in deze workflow.

---

## Fase 1: Research

Voer eerst de research-stap uit. Lees en volg: `../Research.md`

Vergelijk de gevonden versie met de draaiende versie:

```bash
ssh root@$IP "docker inspect openclaw-gateway --format '{{.Config.Image}}'"
```

---

## Fase 2: Status-check

Voer alle checks uit en presenteer de uitkomsten aan de gebruiker.

### Containers

```bash
ssh root@<IP> "docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Image}}'"
```

Verwacht: `openclaw-gateway` en `openclaw-caddy` (als Caddy geïnstalleerd) draaien.

### Firewall

```bash
# Hetzner Cloud Firewall
hcloud firewall list
hcloud firewall describe <naam-firewall>

# UFW op de VPS
ssh root@<IP> "ufw status verbose"
```

### Disk en geheugen

```bash
ssh root@<IP> "df -h / && free -h && docker stats --no-stream"
```

### Reboot nodig?

```bash
ssh root@<IP> "test -f /var/run/reboot-required && echo 'REBOOT AANBEVOLEN' || echo 'Geen reboot nodig'"
```

### Backups

```bash
ssh root@<IP> "ls -lh /root/backups/openclaw/ 2>/dev/null | tail -5 || echo 'Geen backups gevonden'"
```

---

## Fase 3: Backup uitvoeren

Maak altijd een backup vóór een update.

```bash
# Op de VPS: online backup van SQLite databases (WAL-safe, geen container-stop nodig)
DATE=$(date +%Y%m%d-%H%M)
ssh root@$IP << EOF
mkdir -p /root/backups/openclaw
sqlite3 /root/.openclaw/memory/main.sqlite ".backup '/root/backups/openclaw/main-$DATE.sqlite'"
sqlite3 /root/.openclaw/memory/coach.sqlite ".backup '/root/backups/openclaw/coach-$DATE.sqlite'"
cp /root/.openclaw/openclaw.json /root/backups/openclaw/openclaw-$DATE.json

# Integriteitscontrole: verifieer dat de backup bruikbaar is
sqlite3 /root/backups/openclaw/main-$DATE.sqlite "PRAGMA integrity_check;"
sqlite3 /root/backups/openclaw/coach-$DATE.sqlite "PRAGMA integrity_check;"
# Verwacht: "ok" voor beide databases

ls -lh /root/backups/openclaw/ | tail -6
EOF
```

Lokale kopie ophalen (optioneel maar aanbevolen):

```bash
scp root@<IP>:/root/backups/openclaw/main-$DATE.sqlite ~/Downloads/
scp root@<IP>:/root/backups/openclaw/coach-$DATE.sqlite ~/Downloads/
```

---

## Fase 4: Update (als nieuwe versie beschikbaar)

Alleen uitvoeren als research een nieuwere versie heeft gevonden.

```bash
# Stap 1: Huidige image taggen als rollback
ssh root@<IP> << 'EOF'
CURRENT=$(docker inspect openclaw-gateway --format '{{.Config.Image}}' 2>/dev/null || echo "openclaw:local")
docker tag "$CURRENT" "openclaw:rollback-$(date +%Y%m%d)"
echo "Rollback tag aangemaakt"
EOF

# Stap 2: Nieuwe versie bouwen
ssh root@<IP> << 'EOF'
cd /root/openclaw-src
git pull origin main
docker build -t openclaw:local .
echo "Build klaar"
EOF

# Stap 3: Deployen
ssh root@<IP> << 'EOF'
cd /root/openclaw
docker compose up -d
docker compose logs --tail=20 openclaw-gateway
EOF
```

### Rollback (als de update mis gaat)

```bash
# Zoek eerst de exacte rollback-tagnaam op — het kan een andere datum zijn:
ssh root@$IP "docker images | grep rollback"

# Gebruik de tagnaam uit de output hierboven:
ROLLBACK_TAG="openclaw:rollback-<datum>"   # ← vul in uit bovenstaande output

ssh root@$IP << EOF
cd /root/openclaw
docker tag $ROLLBACK_TAG openclaw:local
docker compose up -d
docker compose logs --tail=10 openclaw-gateway
EOF
```

---

## Fase 5: Verificatie na onderhoud

Doorloop de snelle checks uit `../Runbooks/06-Verify.md` (sectie "Quick check").

---

## Automatische backup instellen (eenmalig)

Als er nog geen automatische backup-cron actief is:

```bash
# Backup-script aanmaken
ssh root@<IP> "cat > /root/backup-openclaw.sh" << 'SCRIPT'
#!/bin/bash
set -euo pipefail
DATE=$(date +%Y%m%d-%H%M)
BACKUP_DIR="/root/backups/openclaw"
mkdir -p "$BACKUP_DIR"
sqlite3 /root/.openclaw/memory/main.sqlite ".backup '$BACKUP_DIR/main-$DATE.sqlite'"
sqlite3 /root/.openclaw/memory/coach.sqlite ".backup '$BACKUP_DIR/coach-$DATE.sqlite'"
cp /root/.openclaw/openclaw.json "$BACKUP_DIR/openclaw-$DATE.json"
# Bewaar laatste 14 backups per bestandstype
cd "$BACKUP_DIR"
ls -t main-*.sqlite 2>/dev/null | tail -n +15 | xargs -r rm
ls -t coach-*.sqlite 2>/dev/null | tail -n +15 | xargs -r rm
ls -t openclaw-*.json 2>/dev/null | tail -n +15 | xargs -r rm
echo "[$DATE] Backup klaar"
SCRIPT

ssh root@<IP> "chmod +x /root/backup-openclaw.sh"

# Cron: dagelijks 03:00 UTC
ssh root@<IP> "(crontab -l 2>/dev/null; echo '0 3 * * * /root/backup-openclaw.sh >> /root/backups/backup.log 2>&1') | crontab -"
ssh root@<IP> "crontab -l"
```

---

## Maandelijkse checklist

- [ ] Research uitgevoerd (versie + security advisories)
- [ ] Containers draaien zonder errors
- [ ] Firewall-regels intact (Hetzner + UFW)
- [ ] Disk niet vol (< 80% gebruik)
- [ ] Backup gemaakt en geverifieerd
- [ ] Update uitgevoerd indien nieuwe versie beschikbaar
- [ ] Reboot uitgevoerd indien aanbevolen
- [ ] Docker images opgeruimd: `ssh root@<IP> "docker image prune -f"`
