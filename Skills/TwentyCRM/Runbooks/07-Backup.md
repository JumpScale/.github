# Stap 7: Backup procedure

## Doel
Dagelijkse automatische backups van Twenty CRM database en n8n data.

---

## 7.1 Backup script aanmaken [Claude doet dit]

```bash
cat > /opt/twenty-crm/backup.sh << 'SCRIPT'
#!/bin/bash
BACKUP_DIR="/opt/backups"
DATE=$(date +%Y%m%d-%H%M)
mkdir -p "$BACKUP_DIR"

echo "=== Backup started at $(date) ==="

# Twenty CRM Postgres backup
echo "Backing up Twenty CRM database..."
docker exec twenty-db-1 pg_dumpall -U postgres | gzip > "$BACKUP_DIR/twenty-db-$DATE.sql.gz"
echo "  -> twenty-db-$DATE.sql.gz ($(du -h "$BACKUP_DIR/twenty-db-$DATE.sql.gz" | cut -f1))"

# n8n data backup (indien aanwezig)
if docker ps --format '{{.Names}}' | grep -q n8n; then
    echo "Backing up n8n data..."
    docker cp n8n-n8n-1:/home/node/.n8n "$BACKUP_DIR/n8n-data-$DATE"
    tar -czf "$BACKUP_DIR/n8n-data-$DATE.tar.gz" -C "$BACKUP_DIR" "n8n-data-$DATE"
    rm -rf "$BACKUP_DIR/n8n-data-$DATE"
    echo "  -> n8n-data-$DATE.tar.gz ($(du -h "$BACKUP_DIR/n8n-data-$DATE.tar.gz" | cut -f1))"
fi

# Cleanup: keep last 7 days
find "$BACKUP_DIR" -name "twenty-db-*.sql.gz" -mtime +7 -delete
find "$BACKUP_DIR" -name "n8n-data-*.tar.gz" -mtime +7 -delete

echo "=== Backup completed at $(date) ==="
SCRIPT
chmod +x /opt/twenty-crm/backup.sh
```

## 7.2 Test de backup [Claude doet dit]

```bash
/opt/twenty-crm/backup.sh
ls -lh /opt/backups/
```

Controleer dat de bestanden aangemaakt zijn en een redelijke grootte hebben (niet 0 bytes).

## 7.3 Dagelijkse cron instellen [Claude doet dit]

```bash
(crontab -l 2>/dev/null | grep -v backup.sh; echo "0 3 * * * /opt/twenty-crm/backup.sh >> /var/log/backup.log 2>&1") | crontab -
```

Dit draait de backup elke dag om 03:00 UTC.

## 7.4 Restore procedure

### Twenty CRM database restoren

```bash
# 1. Stop Twenty CRM
cd /opt/twenty-crm && docker compose down

# 2. Start alleen de database
docker compose up -d db
sleep 5

# 3. Restore
gunzip -c /opt/backups/twenty-db-YYYYMMDD-HHMM.sql.gz | docker exec -i twenty-db-1 psql -U postgres

# 4. Start alles
docker compose up -d
```

### n8n data restoren

```bash
# 1. Stop n8n
cd /opt/n8n && docker compose down

# 2. Restore
tar -xzf /opt/backups/n8n-data-YYYYMMDD-HHMM.tar.gz -C /tmp/
docker cp /tmp/n8n-data-YYYYMMDD-HHMM/. n8n-n8n-1:/home/node/.n8n/

# 3. Start n8n
docker compose up -d
```

---

## Verificatie

```bash
crontab -l | grep backup              # Cron ingesteld
ls -lh /opt/backups/                   # Recente bestanden aanwezig
cat /var/log/backup.log | tail -20     # Geen errors in log
```
