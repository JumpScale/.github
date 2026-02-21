# Install Workflow

Begeleide installatie van Trigger.dev op een eigen server met Docker Compose.

```
We gaan Trigger.dev installeren op je server. Ik loop eerst even langs alles
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

Trigger.dev vereist minimaal:
- 8 GB RAM (aanbevolen: 16 GB)
- 2 CPU cores
- 40 GB disk

### Check 3 -- Domeinnaam [Jij moet dit doen]

```
Heb je een (sub)domein klaar voor Trigger.dev?
Bijvoorbeeld: trigger.jouwbedrijf.nl

Dit is nodig voor SSL en voor de API URL die je SDK gebruikt.
```

### Check 4 -- Vrije poorten [Claude doet dit]

```bash
ss -tlnp | grep -E ':80|:443|:3040'
```

### Samenvatting

```
Vereisten-check klaar:
- SSH toegang tot server
- Docker + Docker Compose aanwezig
- RAM: [hoeveelheid] (minimaal 8 GB)
- Domeinnaam: [domein]
- Poorten beschikbaar

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

A) Alleen Trigger.dev (fresh start, dashboard + API)
B) Trigger.dev + project setup (SDK configuratie, eerste task deployen)
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
- Trigger.dev deployen (Docker Compose)
- Reverse proxy + SSL
[als B] - SDK configuratie + eerste task
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

### Stap 2 -- Trigger.dev installeren (altijd)
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
- Trigger.dev dashboard: https://[domein]
- API URL (voor SDK): https://[domein]
- Backup: dagelijks, 7 dagen retentie

Bewaar deze gegevens veilig:
- Server IP
- Trigger.dev login e-mailadres (magic link auth)
- MAGIC_LINK_SECRET en SESSION_SECRET (in /opt/trigger/.env)

Volgende stappen:
- Periodiek onderhoud: zeg "voer onderhoud uit op mijn Trigger.dev"
- SDK configureren in je project: TRIGGER_API_URL=https://[domein]
- Eerste task deployen: npx trigger.dev@latest deploy
```
