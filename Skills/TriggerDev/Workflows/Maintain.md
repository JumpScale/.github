# Maintain Workflow

Periodiek onderhoud voor een bestaande Trigger.dev v4 self-hosted installatie.
Aanbevolen: eens per maand uitvoeren, of na een nieuwe Trigger.dev release.

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
ssh root@<SERVER_IP> "docker inspect trigger-webapp --format '{{.Config.Image}}'"
```

---

## Fase 2: Status-check

### Containers (verwacht: 9 stuks)

```bash
ssh root@<SERVER_IP> "/opt/trigger-dev/trigger-ctl.sh status"
```

Verwacht: webapp, postgres, redis, electric, clickhouse, minio, supervisor, docker-proxy en evt. registry draaien.

### Disk en geheugen

```bash
ssh root@<SERVER_IP> "df -h / && free -h && docker stats --no-stream"
```

### SSL certificaat

```bash
ssh root@<SERVER_IP> "certbot certificates 2>/dev/null || echo 'certbot niet geinstalleerd'"
```

### Backups

```bash
ssh root@<SERVER_IP> "ls -lh /opt/trigger-dev/backups/ 2>/dev/null | tail -5 || echo 'Geen backups gevonden'"
```

### Reboot nodig?

```bash
ssh root@<SERVER_IP> "test -f /var/run/reboot-required && echo 'REBOOT AANBEVOLEN' || echo 'Geen reboot nodig'"
```

### ghcr.io login geldig?

```bash
ssh root@<SERVER_IP> "docker pull ghcr.io/triggerdotdev/trigger.dev:v4-beta --quiet && echo 'ghcr.io login OK' || echo 'ghcr.io login VERLOPEN - docker login ghcr.io uitvoeren'"
```

### Samenvatting

```
Status-check:
- Containers: [status] (verwacht: 9 draaien)
- Disk: [gebruik]%
- SSL: geldig tot [datum]
- Backups: [laatste datum]
- Reboot: [wel/niet nodig]
- ghcr.io: [geldig/verlopen]

[Als nieuwe versie beschikbaar:] Update beschikbaar: [huidige] -> [nieuwe versie]
```

---

## Fase 3: Backup

Maak altijd een backup voor een update.

```bash
ssh root@<SERVER_IP> << 'EOF'
DATE=$(date +%Y%m%d-%H%M)
BACKUP_DIR="/opt/trigger-dev/backups"
mkdir -p "$BACKUP_DIR"

# Database backup (container: trigger-postgres-1, user: postgres)
docker exec trigger-postgres-1 pg_dumpall -U postgres | gzip > "$BACKUP_DIR/trigger-db-$DATE.sql.gz"

echo "Backup klaar: $BACKUP_DIR"
ls -lh "$BACKUP_DIR" | tail -5
EOF
```

---

## Fase 4: Update (als nieuwe versie beschikbaar)

Alleen uitvoeren als research een nieuwere versie heeft gevonden.

```bash
ssh root@<SERVER_IP> << 'EOF'
# Huidige images noteren voor rollback
CURRENT=$(docker inspect trigger-webapp --format '{{.Config.Image}}')
echo "Huidige image: $CURRENT"

# Update image tag in .env als nodig
# Voorbeeld: sed -i 's/TRIGGER_IMAGE_TAG=v4-beta/TRIGGER_IMAGE_TAG=v4/' /opt/trigger-dev/.env

# Nieuwe versie pullen
docker compose -f /opt/trigger-dev/webapp/docker-compose.yml --env-file /opt/trigger-dev/.env pull
docker compose -f /opt/trigger-dev/worker/docker-compose.yml --env-file /opt/trigger-dev/.env pull

# Deployen via management script
/opt/trigger-dev/trigger-ctl.sh restart

# Logs checken
/opt/trigger-dev/trigger-ctl.sh logs webapp 2>&1 | tail -20
EOF
```

### Rollback (als de update mis gaat)

```bash
ssh root@<SERVER_IP> << 'EOF'
# Pas de TRIGGER_IMAGE_TAG in /opt/trigger-dev/.env terug naar de vorige versie
# Voorbeeld: sed -i 's/TRIGGER_IMAGE_TAG=v4/TRIGGER_IMAGE_TAG=v4-beta/' /opt/trigger-dev/.env

/opt/trigger-dev/trigger-ctl.sh restart
/opt/trigger-dev/trigger-ctl.sh logs webapp 2>&1 | tail -20
EOF
```

---

## Fase 5: Opruimen

```bash
ssh root@<SERVER_IP> << 'EOF'
# Oude Docker images opruimen
docker image prune -f

# Oude backups opruimen (bewaar laatste 7)
cd /opt/trigger-dev/backups
ls -t trigger-db-*.sql.gz 2>/dev/null | tail -n +8 | xargs -r rm

echo "Opruimen klaar"
df -h /
EOF
```

---

## Fase 6: Verificatie

Doorloop de checks uit `../Runbooks/06-Verify.md`.

---

## Maandelijkse checklist

- [ ] Research uitgevoerd (versie + security advisories)
- [ ] Alle 9 containers draaien (webapp, postgres, redis, electric, clickhouse, minio, supervisor, docker-proxy, registry)
- [ ] Disk < 80% gebruik
- [ ] SSL certificaat geldig
- [ ] ghcr.io login geldig
- [ ] Backup gemaakt en geverifieerd
- [ ] Update uitgevoerd indien nieuwe versie beschikbaar
- [ ] Reboot uitgevoerd indien aanbevolen
- [ ] Docker images opgeruimd
