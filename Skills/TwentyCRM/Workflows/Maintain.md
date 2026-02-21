# Maintain Workflow

Periodiek onderhoud van een bestaande Twenty CRM installatie.

```
We gaan je Twenty CRM installatie controleren en bijwerken waar nodig.
Ik check alles stap voor stap -- als er iets moet gebeuren, bespreek ik het eerst met je.
```

---

## Fase 1: Research

Lees en volg: `../Research.md`

Presenteer bevindingen aan de gebruiker, met nadruk op:
- Is er een nieuwe versie beschikbaar?
- Zijn er security advisories?

---

## Fase 2: Health checks

Voer alle checks uit en rapporteer resultaten.

### Check 1 -- Container status

```bash
docker compose -f /opt/twenty-crm/docker-compose.yml ps
docker compose -f /opt/n8n/docker-compose.yml ps 2>/dev/null
```

Verwacht: alle containers "Up" en "healthy".

### Check 2 -- Disk usage

```bash
df -h /
du -sh /opt/twenty-crm /opt/n8n /opt/backups 2>/dev/null
```

Waarschuw als disk boven 80% zit.

### Check 3 -- Backups

```bash
ls -lh /opt/backups/ | tail -5
```

Controleer:
- Zijn er recente backups (< 24 uur)?
- Zijn de bestandsgroottes normaal (niet 0 bytes)?

### Check 4 -- SSL certificaten

```bash
certbot certificates
```

Controleer vervaldatum. Waarschuw als < 14 dagen.

### Check 5 -- Security headers

```bash
curl -sI https://[domein] | grep -iE "x-frame|strict-transport|x-content-type|referrer-policy"
```

Controleer of alle security headers aanwezig zijn.

### Check 6 -- Firewall

```bash
ufw status
```

Controleer of alleen noodzakelijke poorten open zijn.

### Check 7 -- Fail2ban

```bash
fail2ban-client status sshd
```

Controleer of fail2ban actief is en hoeveel bans er zijn geweest.

---

## Fase 3: Updates (indien nodig)

### Twenty CRM updaten

Alleen als er een nieuwe versie beschikbaar is EN de gebruiker akkoord geeft.

```bash
# 1. Backup eerst
/opt/twenty-crm/backup.sh

# 2. Pull nieuwe images
cd /opt/twenty-crm && docker compose pull

# 3. Herstart met nieuwe versie
docker compose up -d

# 4. Wacht tot healthy
docker compose ps
```

### n8n updaten

```bash
cd /opt/n8n && docker compose pull && docker compose up -d
```

---

## Fase 4: Samenvatting

```
Onderhoud voltooid.

Status:
- Twenty CRM: [versie] - [status]
- Database: [grootte] - [status]
- Backups: [laatste backup datum] - [status]
- SSL: geldig tot [datum]
- Security: [status]
[als n8n] - n8n: [status]
[als update] - Geupdate naar: [nieuwe versie]

Volgende onderhoud aanbevolen: over 2-4 weken
```
