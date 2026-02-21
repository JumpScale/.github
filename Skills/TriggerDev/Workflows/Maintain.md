# Maintain Workflow

Periodiek onderhoud voor een bestaande Trigger.dev self-hosted installatie.
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
ssh root@<SERVER_IP> "docker compose -f /opt/trigger/docker-compose.yml exec -T webapp cat package.json 2>/dev/null | python3 -c \"import json,sys; print(json.load(sys.stdin).get('version','onbekend'))\" 2>/dev/null || docker inspect trigger-webapp --format '{{.Config.Image}}'"
```

---

## Fase 2: Status-check

### Containers

```bash
ssh root@<SERVER_IP> "docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Image}}' | grep -E 'trigger|postgres|redis|coordinator|provider|electric'"
```

Verwacht: webapp, postgres, redis, docker-provider, coordinator en electric containers draaien.

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
ssh root@<SERVER_IP> "ls -lh /opt/trigger/backups/ 2>/dev/null | tail -5 || echo 'Geen backups gevonden'"
```

### Reboot nodig?

```bash
ssh root@<SERVER_IP> "test -f /var/run/reboot-required && echo 'REBOOT AANBEVOLEN' || echo 'Geen reboot nodig'"
```

### Samenvatting

```
Status-check:
- Containers: [status]
- Disk: [gebruik]%
- SSL: geldig tot [datum]
- Backups: [laatste datum]
- Reboot: [wel/niet nodig]

[Als nieuwe versie beschikbaar:] Update beschikbaar: [huidige] -> [nieuwe versie]
```

---

## Fase 3: Backup

Maak altijd een backup voor een update.

```bash
ssh root@<SERVER_IP> << 'EOF'
DATE=$(date +%Y%m%d-%H%M)
BACKUP_DIR="/opt/trigger/backups"
mkdir -p "$BACKUP_DIR"

# Database backup
docker compose -f /opt/trigger/docker-compose.yml exec -T postgres pg_dumpall -U trigger | gzip > "$BACKUP_DIR/trigger-db-$DATE.sql.gz"

echo "Backup klaar: $BACKUP_DIR"
ls -lh "$BACKUP_DIR" | tail -5
EOF
```

---

## Fase 4: Update (als nieuwe versie beschikbaar)

Alleen uitvoeren als research een nieuwere versie heeft gevonden.

```bash
ssh root@<SERVER_IP> << 'EOF'
cd /opt/trigger

# Huidige images noteren voor rollback
CURRENT=$(docker inspect trigger-webapp --format '{{.Config.Image}}')
echo "Huidige image: $CURRENT"

# Nieuwe versie pullen
docker compose pull

# Deployen
docker compose up -d

# Logs checken
docker compose logs --tail=20 webapp
EOF
```

### Rollback (als de update mis gaat)

```bash
ssh root@<SERVER_IP> << 'EOF'
cd /opt/trigger
# Pas de image tags in docker-compose.yml terug naar de vorige versie
docker compose up -d
docker compose logs --tail=20 webapp
EOF
```

---

## Fase 5: Opruimen

```bash
ssh root@<SERVER_IP> << 'EOF'
# Oude Docker images opruimen
docker image prune -f

# Oude backups opruimen (bewaar laatste 7)
cd /opt/trigger/backups
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
- [ ] Alle containers draaien (webapp, postgres, redis, docker-provider, coordinator, electric)
- [ ] Disk < 80% gebruik
- [ ] SSL certificaat geldig
- [ ] Backup gemaakt en geverifieerd
- [ ] Update uitgevoerd indien nieuwe versie beschikbaar
- [ ] Reboot uitgevoerd indien aanbevolen
- [ ] Docker images opgeruimd
