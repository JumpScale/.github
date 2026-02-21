# 05 -- Backup procedure

## Doel

Automatische dagelijkse backups van Trigger.dev database.

---

## Stap 1: Backup script aanmaken [Claude doet dit]

```bash
ssh root@<SERVER_IP> "cat > /opt/trigger/backup.sh" << 'SCRIPT'
#!/bin/bash
set -euo pipefail

DATE=$(date +%Y%m%d-%H%M)
BACKUP_DIR="/opt/trigger/backups"
mkdir -p "$BACKUP_DIR"

# PostgreSQL backup
docker compose -f /opt/trigger/docker-compose.yml exec -T postgres pg_dumpall -U trigger | gzip > "$BACKUP_DIR/trigger-db-$DATE.sql.gz"

# Retentie: bewaar laatste 7 backups
cd "$BACKUP_DIR"
ls -t trigger-db-*.sql.gz 2>/dev/null | tail -n +8 | xargs -r rm

echo "[$DATE] Backup voltooid"
SCRIPT

ssh root@<SERVER_IP> "chmod +x /opt/trigger/backup.sh"
```

## Stap 2: Cron instellen [Claude doet dit]

```bash
# Dagelijks 03:00 UTC
ssh root@<SERVER_IP> "(crontab -l 2>/dev/null | grep -v backup.sh; echo '0 3 * * * /opt/trigger/backup.sh >> /opt/trigger/backups/backup.log 2>&1') | crontab -"
ssh root@<SERVER_IP> "crontab -l"
```

## Stap 3: Eerste backup testen [Claude doet dit]

```bash
ssh root@<SERVER_IP> "/opt/trigger/backup.sh && ls -lh /opt/trigger/backups/"
```

## Restore procedure

### Database herstellen

```bash
ssh root@<SERVER_IP> << 'EOF'
cd /opt/trigger

# Kies de backup die je wilt herstellen
ls -lh backups/trigger-db-*.sql.gz

# Stop webapp en gerelateerde services (niet de database)
docker compose stop webapp coordinator docker-provider electric

# Restore
gunzip -c backups/trigger-db-<DATUM>.sql.gz | docker compose exec -T postgres psql -U trigger

# Start alles
docker compose up -d
EOF
```

## Verificatie

```
Backup check:
- [ ] Backup script bestaat en is executable
- [ ] Cron draait dagelijks om 03:00 UTC
- [ ] Eerste handmatige backup geslaagd
- [ ] Backup bestanden aanwezig in /opt/trigger/backups/
```
