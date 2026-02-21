# Install Workflow

Begeleide installatie van Trigger.dev v4 op een eigen server met Docker Compose.

```
We gaan Trigger.dev v4 installeren op je server. Dit is een grotere setup dan v3:
9 containers in twee Docker Compose stacks. Ik loop eerst even langs alles
wat we nodig hebben -- wat ik zelf kan checken doe ik meteen, en als ik
jou ergens bij nodig heb, zeg ik het duidelijk. Klaar?
```

---

## Fase 0: Vereisten checken

Label elke stap: **[Claude doet dit]** of **[Jij moet dit doen]**.

### Check 1 -- SSH toegang [Claude doet dit]

```bash
ssh root@<SERVER_IP> "uname -a && docker --version && docker compose version"
```

- **Werkt:** Noteer OS en Docker versie. Ga door.
- **Docker niet geinstalleerd:** Installeer via `curl -fsSL https://get.docker.com | sh`

### Check 2 -- Minimale specs [Claude doet dit]

```bash
ssh root@<SERVER_IP> "free -h && nproc && df -h /"
```

Trigger.dev v4 vereist minimaal:
- 8 GB RAM (aanbevolen: 16 GB) -- ClickHouse en MinIO vragen meer resources dan v3
- 2 CPU cores
- 40 GB disk

Bij een shared VPS met beperkt RAM: stel `NODE_MAX_OLD_SPACE_SIZE=1600` in.

### Check 3 -- Domeinnaam [Jij moet dit doen]

```
Heb je een (sub)domein klaar voor Trigger.dev?
Bijvoorbeeld: trigger.jouwbedrijf.nl

Dit is nodig voor SSL en voor de API URL die je SDK gebruikt.
```

### Check 4 -- Vrije poorten [Claude doet dit]

```bash
ss -tlnp | grep -E ':80|:443|:8030'
```

### Check 5 -- GitHub Container Registry [Jij moet dit doen]

```
Heb je een GitHub account met een Personal Access Token (PAT) dat
write:packages scope heeft? Dit is nodig voor de Docker registry.

v4 gebruikt ghcr.io voor het opslaan van gebouwde task images.
Een lokale Docker registry werkt niet goed door BuildKit networking issues.
```

### Samenvatting

```
Vereisten-check klaar:
- SSH toegang tot server
- Docker + Docker Compose aanwezig
- RAM: [hoeveelheid] (minimaal 8 GB, aanbevolen 16 GB)
- Domeinnaam: [domein]
- Poorten beschikbaar
- GitHub PAT met write:packages scope

We kunnen beginnen.
```

---

## Fase 1: Research

Lees en volg: `../Research.md`

---

## Fase 2: Keuzemenu

### Vraag 1: Welke combinatie?

```
Wat wil je installeren?

A) Alleen Trigger.dev v4 (fresh start, dashboard + API + 9 containers)
B) Trigger.dev v4 + project setup (SDK configuratie, eerste task deployen)
```

### Vraag 2: Tailscale?

```
Wil je het Trigger.dev dashboard beveiligen met Tailscale VPN?

Ja  -> Dashboard alleen via prive-netwerk. API endpoints blijven publiek.
Nee -> Publiek bereikbaar met login + security headers.
```

### Bevestiging

```
Installatie-plan:
- Trigger.dev v4 deployen (2 Docker Compose stacks, 9 containers)
- Reverse proxy + SSL (nginx + certbot)
[als B] - SDK configuratie + eerste task deploy (vanaf VPS)
[als Tailscale=ja] - Tailscale installeren
- Security hardening
- Backup procedure
- Verificatie

Doorgaan?
```

---

## Fase 3: Uitvoering

### Stap 1 -- Server voorbereiden (altijd)
Lees en volg: `../Runbooks/01-Vps.md`

### Stap 2 -- Trigger.dev v4 installeren (altijd)
Lees en volg: `../Runbooks/02-TriggerDev.md`

### Stap 3 -- Reverse proxy + SSL (altijd)
Lees en volg: `../Runbooks/03-ReverseProxy.md`

### Stap 4 -- Security hardening (altijd)
Lees en volg: `../Runbooks/04-Security.md`

### Stap 5 -- Backup procedure (altijd)
Lees en volg: `../Runbooks/05-Backup.md`

---

## Fase 4: Verificatie

Lees en volg: `../Runbooks/06-Verify.md`

---

## Afsluiting

```
Installatie voltooid.

Samenvatting:
- Trigger.dev v4 dashboard: https://[domein]
- API URL (voor SDK): https://[domein]
- Containers: 9 stuks in 2 Docker Compose stacks
- Management: /opt/trigger-dev/trigger-ctl.sh {up|down|restart|logs|status}
- Backup: dagelijks, 7 dagen retentie

Bewaar deze gegevens veilig:
- Server IP
- Trigger.dev login e-mailadres (magic link auth)
- Secrets in /opt/trigger-dev/.env
- GitHub PAT voor ghcr.io

Volgende stappen:
- Periodiek onderhoud: zeg "voer onderhoud uit op mijn Trigger.dev"
- SDK configureren in je project: TRIGGER_API_URL=https://[domein]
- Tasks deployen (vanaf VPS): npx trigger.dev@latest deploy --api-url https://[domein]
```
