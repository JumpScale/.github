# Maintain Workflow

Periodiek onderhoud voor een bestaande n8n self-hosted installatie.
Aanbevolen: eens per maand uitvoeren, of na een nieuwe n8n release.

---

## Fase 0: Verbinding opzetten

Vraag de gebruiker naar het IP-adres of de domeinnaam van de server:

```bash
ssh root@<SERVER_IP> "uname -a && docker --version"
```

---

## Fase 1: Research

Lees en volg: `../Research.md`

Vergelijk de gevonden versie met de draaiende versie:

```bash
ssh root@<SERVER_IP> "docker compose -f /opt/n8n/docker-compose.yml exec -T n8n n8n --version"
```

---

## Fase 2: Status-check

### Containers

```bash
ssh root@<SERVER_IP> "docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Image}}'"
```

Verwacht: `n8n` container draait.

### Disk en geheugen

```bash
ssh root@<SERVER_IP> "df -h / && free -h && docker stats --no-stream"
```

### Workflows status

```bash
ssh root@<SERVER_IP> "docker compose -f /opt/n8n/docker-compose.yml exec -T n8n n8n list:workflow"
```

### SSL certificaat

```bash
ssh root@<SERVER_IP> "certbot certificates 2>/dev/null || echo 'certbot niet geinstalleerd'"
```

### Backups

```bash
ssh root@<SERVER_IP> "ls -lh /opt/n8n/backups/ 2>/dev/null | tail -5 || echo 'Geen backups gevonden'"
```

### Reboot nodig?

```bash
ssh root@<SERVER_IP> "test -f /var/run/reboot-required && echo 'REBOOT AANBEVOLEN' || echo 'Geen reboot nodig'"
```

### Samenvatting

```
Status-check:
✓ Containers: [status]
✓ Disk: [gebruik]%
✓ Workflows: [aantal] actief
✓ SSL: geldig tot [datum]
✓ Backups: [laatste datum]
✓ Reboot: [wel/niet nodig]

[Als nieuwe versie beschikbaar:] Update beschikbaar: [huidige] -> [nieuwe versie]
```

---

## Fase 3: Backup

Maak altijd een backup voor een update.

```bash
ssh root@<SERVER_IP> << 'EOF'
DATE=$(date +%Y%m%d-%H%M)
BACKUP_DIR="/opt/n8n/backups"
mkdir -p "$BACKUP_DIR"

# Database backup
docker compose -f /opt/n8n/docker-compose.yml exec -T n8n-db pg_dumpall -U n8n | gzip > "$BACKUP_DIR/n8n-db-$DATE.sql.gz"

# n8n data (workflows, credentials encrypted)
docker cp n8n:/home/node/.n8n "$BACKUP_DIR/n8n-data-$DATE"
tar czf "$BACKUP_DIR/n8n-data-$DATE.tar.gz" -C "$BACKUP_DIR" "n8n-data-$DATE"
rm -rf "$BACKUP_DIR/n8n-data-$DATE"

echo "Backup klaar: $BACKUP_DIR"
ls -lh "$BACKUP_DIR" | tail -5
EOF
```

---

## Fase 4: Update (als nieuwe versie beschikbaar)

Alleen uitvoeren als research een nieuwere versie heeft gevonden.

```bash
ssh root@<SERVER_IP> << 'EOF'
cd /opt/n8n

# Huidige image noteren voor rollback
CURRENT=$(docker inspect n8n --format '{{.Config.Image}}')
echo "Huidige image: $CURRENT"

# Nieuwe versie pullen
docker compose pull

# Deployen
docker compose up -d

# Versie controleren
docker compose exec -T n8n n8n --version

# Logs checken
docker compose logs --tail=20 n8n
EOF
```

### Rollback (als de update mis gaat)

```bash
ssh root@<SERVER_IP> << 'EOF'
cd /opt/n8n
# Pas de image tag in docker-compose.yml terug naar de vorige versie
# Bijvoorbeeld: n8nio/n8n:1.76.1 -> n8nio/n8n:1.75.0
docker compose up -d
docker compose logs --tail=20 n8n
EOF
```

---

## Fase 5: Opruimen

```bash
ssh root@<SERVER_IP> << 'EOF'
# Oude Docker images opruimen
docker image prune -f

# Oude backups opruimen (bewaar laatste 7)
cd /opt/n8n/backups
ls -t n8n-db-*.sql.gz 2>/dev/null | tail -n +8 | xargs -r rm
ls -t n8n-data-*.tar.gz 2>/dev/null | tail -n +8 | xargs -r rm

echo "Opruimen klaar"
df -h /
EOF
```

---

## Fase 6: Verificatie

Doorloop de checks uit `../Runbooks/08-Verify.md`.

---

## Maandelijkse checklist

- [ ] Research uitgevoerd (versie + security advisories)
- [ ] Containers draaien
- [ ] Disk < 80% gebruik
- [ ] SSL certificaat geldig
- [ ] Backup gemaakt en geverifieerd
- [ ] Update uitgevoerd indien nieuwe versie beschikbaar
- [ ] Reboot uitgevoerd indien aanbevolen
- [ ] Docker images opgeruimd
