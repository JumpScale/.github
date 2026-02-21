# Install Workflow

Begeleide installatie van n8n op een eigen server met Docker Compose.

```
We gaan n8n installeren op je server. Ik loop eerst even langs alles
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

### Check 2 -- Domeinnaam [Jij moet dit doen]

```
Heb je een (sub)domein klaar voor n8n?
Bijvoorbeeld: n8n.jouwbedrijf.nl

Dit is nodig voor SSL en voor de webhook URLs die externe services gebruiken.
```

### Check 3 -- Vrije poorten [Claude doet dit]

```bash
ss -tlnp | grep -E ':80|:443|:5678'
```

### Samenvatting

```
Vereisten-check klaar:
✓ SSH toegang tot server
✓ Docker + Docker Compose aanwezig
✓ Domeinnaam: [domein]
✓ Poorten beschikbaar

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

A) Alleen n8n (fresh start)
B) n8n + migratie van cloud workflows
C) n8n + migratie + credentials begeleiding
```

### Vraag 2: Reverse proxy?

```
Welke reverse proxy?

1) nginx (betrouwbaar, wijd verspreid)
2) Caddy (automatische SSL, minder configuratie)
```

### Vraag 3: Tailscale?

```
Wil je de n8n editor beveiligen met Tailscale VPN?

Ja  -> Editor alleen via prive-netwerk. Webhooks blijven publiek.
Nee -> Publiek bereikbaar met login + security headers.
```

### Bevestiging

```
Installatie-plan:
✓ n8n deployen (Docker Compose)
✓ Reverse proxy + SSL
[als B of C] ✓ Workflows migreren van cloud
[als C] ✓ Credentials begeleiding
[als Tailscale=ja] ✓ Tailscale installeren
✓ Security hardening
✓ Backup procedure
✓ Verificatie

Doorgaan?
```

---

## Fase 3: Uitvoering

### Stap 1 -- Server voorbereiden (altijd)
Lees en volg: `../Runbooks/01-Vps.md`

### Stap 2 -- n8n installeren (altijd)
Lees en volg: `../Runbooks/02-N8n.md`

### Stap 3 -- Reverse proxy + SSL (altijd)
Lees en volg: `../Runbooks/03-ReverseProxy.md`

### Stap 4 -- Workflows migreren (alleen als B of C)
Lees en volg: `../Runbooks/04-Migration.md`

### Stap 5 -- Credentials instellen (alleen als C)
Lees en volg: `../Runbooks/05-Credentials.md`

### Stap 6 -- Security hardening (altijd)
Lees en volg: `../Runbooks/06-Security.md`

### Stap 7 -- Backup procedure (altijd)
Lees en volg: `../Runbooks/07-Backup.md`

---

## Fase 4: Verificatie

Lees en volg: `../Runbooks/08-Verify.md`

---

## Afsluiting

```
Installatie voltooid.

Samenvatting:
- n8n: https://[domein]
- Webhook base URL: https://[domein]/webhook/
- Backup: dagelijks, 7 dagen retentie

Bewaar deze gegevens veilig:
- Server IP
- n8n login-gegevens

Volgende stappen:
- Periodiek onderhoud: zeg "voer onderhoud uit op mijn n8n"
- Credentials instellen voor je workflows
- Webhook URLs configureren in je externe services
```
