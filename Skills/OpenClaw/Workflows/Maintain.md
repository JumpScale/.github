# Maintain Workflow

Periodiek onderhoud voor een bestaande OpenClaw-installatie.
Aanbevolen: eens per maand uitvoeren, of na een nieuwe OpenClaw-release.

---

## Fase 0: Verbinding opzetten

Vraag de gebruiker naar het IP-adres of de hostnaam van de server.

```bash
ssh root@<IP> "docker ps --format 'table {{.Names}}\t{{.Status}}' | grep openclaw"
```

---

## Fase 1: Research

Voer eerst de research-stap uit. Lees en volg: `../Research.md`

Vergelijk de gevonden versie met de draaiende versie:

```bash
ssh root@<IP> "docker inspect openclaw-gateway --format '{{.Config.Image}}'"
```

---

## Fase 2: Status-check

### Containers

```bash
ssh root@<IP> "docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Image}}'"
```

### Disk en geheugen

```bash
ssh root@<IP> "df -h / && free -h && docker stats --no-stream --format 'table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}' | grep openclaw"
```

### Backups

```bash
ssh root@<IP> "ls -lh /root/backups/openclaw/ 2>/dev/null | tail -5 || echo 'Geen backups gevonden'"
```

### Logs

```bash
ssh root@<IP> "docker logs openclaw-gateway --tail=20 --since=24h 2>&1 | grep -iE 'error|warn|fatal' || echo 'Geen errors in laatste 24 uur'"
```

---

## Fase 3: Backup uitvoeren

Maak altijd een backup voor een update.

```bash
DATE=$(date +%Y%m%d-%H%M)
ssh root@<IP> << EOF
mkdir -p /root/backups/openclaw
sqlite3 /root/.openclaw/memory/main.sqlite ".backup '/root/backups/openclaw/main-$DATE.sqlite'"
sqlite3 /root/.openclaw/memory/coach.sqlite ".backup '/root/backups/openclaw/coach-$DATE.sqlite'"
cp /root/.openclaw/openclaw.json /root/backups/openclaw/openclaw-$DATE.json

# Integriteitscontrole
sqlite3 /root/backups/openclaw/main-$DATE.sqlite "PRAGMA integrity_check;"
sqlite3 /root/backups/openclaw/coach-$DATE.sqlite "PRAGMA integrity_check;"

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
ssh root@<IP> "docker images | grep rollback"

# Gebruik de tagnaam uit de output:
ssh root@<IP> << 'EOF'
cd /root/openclaw
docker tag openclaw:rollback-<datum> openclaw:local
docker compose up -d
docker compose logs --tail=10 openclaw-gateway
EOF
```

---

## Fase 5: Verificatie na onderhoud

Doorloop de snelle checks uit `../Runbooks/04-Verify.md` (sectie "Quick check").

---

## Automatische backup instellen (eenmalig)

Als er nog geen automatische backup-cron actief is:

```bash
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
- [ ] Backup gemaakt en geverifieerd
- [ ] Update uitgevoerd indien nieuwe versie beschikbaar
- [ ] Docker images opgeruimd: `docker image prune -f`
