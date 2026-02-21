# 07 -- Backup procedure

## Doel

Automatische dagelijkse backups van n8n database en data.

---

## Stap 1: Backup script aanmaken [Claude doet dit]

```bash
ssh root@<SERVER_IP> "cat > /opt/n8n/backup.sh" << 'SCRIPT'
#!/bin/bash
set -euo pipefail

DATE=$(date +%Y%m%d-%H%M)
BACKUP_DIR="/opt/n8n/backups"
mkdir -p "$BACKUP_DIR"

# PostgreSQL backup
docker compose -f /opt/n8n/docker-compose.yml exec -T n8n-db pg_dumpall -U n8n | gzip > "$BACKUP_DIR/n8n-db-$DATE.sql.gz"

# n8n data directory (bevat encrypted credentials, settings)
docker cp n8n:/home/node/.n8n "$BACKUP_DIR/n8n-data-$DATE"
tar czf "$BACKUP_DIR/n8n-data-$DATE.tar.gz" -C "$BACKUP_DIR" "n8n-data-$DATE"
rm -rf "$BACKUP_DIR/n8n-data-$DATE"

# Retentie: bewaar laatste 7 backups
cd "$BACKUP_DIR"
ls -t n8n-db-*.sql.gz 2>/dev/null | tail -n +8 | xargs -r rm
ls -t n8n-data-*.tar.gz 2>/dev/null | tail -n +8 | xargs -r rm

echo "[$DATE] Backup voltooid"
SCRIPT

ssh root@<SERVER_IP> "chmod +x /opt/n8n/backup.sh"
```

## Stap 2: Cron instellen [Claude doet dit]

```bash
# Dagelijks 03:00 UTC
ssh root@<SERVER_IP> "(crontab -l 2>/dev/null | grep -v backup.sh; echo '0 3 * * * /opt/n8n/backup.sh >> /opt/n8n/backups/backup.log 2>&1') | crontab -"
ssh root@<SERVER_IP> "crontab -l"
```

## Stap 3: Eerste backup testen [Claude doet dit]

```bash
ssh root@<SERVER_IP> "/opt/n8n/backup.sh && ls -lh /opt/n8n/backups/"
```

## Restore procedure

### Database herstellen

```bash
ssh root@<SERVER_IP> << 'EOF'
cd /opt/n8n

# Kies de backup die je wilt herstellen
ls -lh backups/n8n-db-*.sql.gz

# Stop n8n (niet de database)
docker compose stop n8n

# Restore
gunzip -c backups/n8n-db-<DATUM>.sql.gz | docker compose exec -T n8n-db psql -U n8n

# Start n8n
docker compose up -d
EOF
```

### Data directory herstellen

```bash
ssh root@<SERVER_IP> << 'EOF'
cd /opt/n8n

# Stop alles
docker compose down

# Restore data
tar xzf backups/n8n-data-<DATUM>.tar.gz -C /tmp/
docker cp /tmp/n8n-data-<DATUM>/. n8n:/home/node/.n8n/

# Start
docker compose up -d
EOF
```

## Verificatie

```
Backup check:
- [ ] Backup script bestaat en is executable
- [ ] Cron draait dagelijks om 03:00 UTC
- [ ] Eerste handmatige backup geslaagd
- [ ] Backup bestanden aanwezig in /opt/n8n/backups/
```
