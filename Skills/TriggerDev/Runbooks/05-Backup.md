# 05 -- Backup procedure

## Doel

Automatische dagelijkse backups van Trigger.dev database.

---

## Stap 1: Backup script aanmaken [Claude doet dit]

Trigger.dev v4 backup is toegevoegd aan het bestaande backup script op `/opt/twenty-crm/backup.sh`.

De container heet `trigger-postgres-1` en de database user is `postgres`.

Als er een apart backup script nodig is:

```bash
ssh root@<SERVER_IP> "cat > /opt/trigger-dev/backup.sh" << 'SCRIPT'
#!/bin/bash
set -euo pipefail

DATE=$(date +%Y%m%d-%H%M)
BACKUP_DIR="/opt/trigger-dev/backups"
mkdir -p "$BACKUP_DIR"

# PostgreSQL backup (container: trigger-postgres-1, user: postgres)
docker exec trigger-postgres-1 pg_dumpall -U postgres | gzip > "$BACKUP_DIR/trigger-db-$DATE.sql.gz"

# Retentie: bewaar laatste 7 backups
cd "$BACKUP_DIR"
ls -t trigger-db-*.sql.gz 2>/dev/null | tail -n +8 | xargs -r rm

echo "[$DATE] Backup voltooid"
SCRIPT

ssh root@<SERVER_IP> "chmod +x /opt/trigger-dev/backup.sh"
```

## Stap 2: Cron instellen [Claude doet dit]

Als de backup NIET via het Twenty CRM backup script draait:

```bash
# Dagelijks 03:00 UTC
ssh root@<SERVER_IP> "(crontab -l 2>/dev/null | grep -v trigger-dev/backup.sh; echo '0 3 * * * /opt/trigger-dev/backup.sh >> /opt/trigger-dev/backups/backup.log 2>&1') | crontab -"
ssh root@<SERVER_IP> "crontab -l"
```

## Stap 3: Eerste backup testen [Claude doet dit]

```bash
ssh root@<SERVER_IP> "/opt/trigger-dev/backup.sh && ls -lh /opt/trigger-dev/backups/"
```

## Restore procedure

### Database herstellen

```bash
ssh root@<SERVER_IP> << 'EOF'
cd /opt/trigger-dev

# Kies de backup die je wilt herstellen
ls -lh backups/trigger-db-*.sql.gz

# Stop webapp en gerelateerde services (niet de database)
/opt/trigger-dev/trigger-ctl.sh down

# Start alleen postgres
docker compose -f /opt/trigger-dev/webapp/docker-compose.yml --env-file /opt/trigger-dev/.env up -d postgres
sleep 5

# Restore
gunzip -c backups/trigger-db-<DATUM>.sql.gz | docker exec -i trigger-postgres-1 psql -U postgres

# Start alles
/opt/trigger-dev/trigger-ctl.sh up
EOF
```

## Verificatie

```
Backup check:
- [ ] Backup script bestaat en is executable
- [ ] Cron draait dagelijks om 03:00 UTC (of via Twenty CRM backup script)
- [ ] Eerste handmatige backup geslaagd
- [ ] Backup bestanden aanwezig in /opt/trigger-dev/backups/
- [ ] Container naam correct: trigger-postgres-1 (niet trigger-postgres)
```
