# Install Workflow

Begeleide installatie van Twenty CRM op een eigen server met Docker Compose.

Begroet de gebruiker en kondig aan dat je begint:

```
We gaan Twenty CRM installeren op je server. Ik loop eerst even langs alles
wat we nodig hebben -- wat ik zelf kan checken doe ik meteen, en als ik
jou ergens bij nodig heb, zeg ik het duidelijk. Klaar?
```

---

## Fase 0: Vereisten checken

Voer elke check uit en rapporteer de uitkomst aan de gebruiker.
Label elke stap duidelijk: **[Claude doet dit]** of **[Jij moet dit doen]**.

---

### Check 1 -- SSH toegang tot server [Claude doet dit]

```bash
ssh root@<SERVER_IP> "uname -a && docker --version && docker compose version"
```

- **Werkt:** Noteer OS en Docker versie. Ga door.
- **Geen toegang:** Vraag de gebruiker om het IP-adres en of SSH op poort 22 openstaat.
- **Docker niet geinstalleerd:**

  ```
  Docker is nog niet geinstalleerd op je server. Dat installeer ik voor je.
  ```
  Installeer Docker via het officiele script:
  ```bash
  curl -fsSL https://get.docker.com | sh
  ```

---

### Check 2 -- Domeinnaam [Jij moet dit doen]

```
Heb je een domeinnaam (of subdomein) klaar voor je CRM?
Bijvoorbeeld: crm.jouwbedrijf.nl

Dit is nodig voor het SSL-certificaat (HTTPS).
```

- **Ja:** Noteer het domein. Vraag of het DNS A-record al naar het server-IP wijst.
- **Nee:**

  ```
  Je hebt een (sub)domein nodig. Ga naar je DNS-provider en maak een A-record aan:
    crm.jouwbedrijf.nl -> <SERVER_IP>

  Het kan tot 30 minuten duren voordat dit actief is.
  Geef een seintje als het klaar is.
  ```

---

### Check 3 -- Vrije poorten [Claude doet dit]

```bash
ss -tlnp | grep -E ':80|:443|:3000|:5678'
```

- **Poorten vrij:** Ga door.
- **Poorten bezet:** Meld welke poort bezet is en door welk proces. Overleg met gebruiker.

---

### Samenvatting vereisten

```
Vereisten-check klaar:
✓ SSH toegang tot server
✓ Docker + Docker Compose aanwezig
✓ Domeinnaam: [domein]
✓ Poorten 80/443 beschikbaar

We kunnen beginnen. Ga je mee?
```

---

## Fase 1: Research

Voer eerst de research-stap uit. Lees en volg: `../Research.md`

Presenteer de uitkomst aan de gebruiker voordat je verdergaat.

---

## Fase 2: Keuzemenu

### Vraag 1: Welke combinatie?

```
Wat wil je installeren?

A) Alleen Twenty CRM (server, database, cache)
B) Twenty CRM + n8n (workflow automation erbij, aanbevolen)
C) Twenty CRM + n8n + Custom Objects (datamodel op maat)
```

### Vraag 2: Reverse proxy?

```
Welke reverse proxy wil je gebruiken?

1) nginx (betrouwbaar, wijd verspreid)
2) Caddy (automatische SSL, minder configuratie)
```

### Vraag 3: Tailscale?

```
Wil je de admin-interfaces beveiligen met Tailscale VPN? (aanbevolen)

Ja  -> CRM en n8n alleen bereikbaar via je prive-netwerk. Webhook URLs blijven publiek.
Nee -> Publiek bereikbaar met login + security headers.
```

### Bevestiging

```
Installatie-plan:
✓ Twenty CRM deployen (server, worker, PostgreSQL 16, Redis 8.2)
✓ Reverse proxy + SSL instellen
[als B of C] ✓ n8n installeren
[als C] ✓ Custom objects en relaties aanmaken
[als Tailscale=ja] ✓ Tailscale installeren
✓ Security hardening (headers, firewall, fail2ban)
✓ Backup procedure instellen (dagelijks, 7 dagen retentie)
✓ Verificatie-checklist nalopen

Doorgaan?
```

---

## Fase 3: Uitvoering

Voer de runbooks uit in volgorde. Sla runbooks over die niet van toepassing zijn.

### Stap 1 -- Server voorbereiden (altijd)
Lees en volg: `../Runbooks/01-Vps.md`

### Stap 2 -- Twenty CRM installeren (altijd)
Lees en volg: `../Runbooks/02-TwentyCRM.md`

### Stap 3 -- Reverse proxy + SSL (altijd)
Lees en volg: `../Runbooks/03-ReverseProxy.md`

### Stap 4 -- n8n installeren (alleen als B of C)
Lees en volg: `../Runbooks/04-n8n.md`

### Stap 5 -- Custom Objects (alleen als C)
Lees en volg: `../Runbooks/05-CustomObjects.md`

### Stap 6 -- Security hardening (altijd)
Lees en volg: `../Runbooks/06-Security.md`

### Stap 7 -- Backup procedure (altijd)
Lees en volg: `../Runbooks/07-Backup.md`

---

## Fase 4: Verificatie

Lees en volg: `../Runbooks/08-Verify.md`

---

## Afsluiting

Na succesvolle verificatie:

```
Installatie voltooid.

Samenvatting:
- Twenty CRM: https://[domein]
[als n8n] - n8n: https://[n8n-domein]
- Backup: dagelijks om 03:00, 7 dagen retentie

Bewaar deze gegevens veilig:
- Server IP
- Twenty CRM login-gegevens
- API key (in /opt/twenty-crm/.api-key)
[als n8n] - n8n login-gegevens

Volgende stappen:
- Periodiek onderhoud: zeg "voer onderhoud uit op mijn Twenty CRM"
- Credentials instellen in n8n (OpenAI, Airtable, etc.)
- Webhook URLs configureren in je formulieren
```
