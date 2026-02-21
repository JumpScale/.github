# Twenty CRM op een eigen server -- Installatie Skill

## Wat is Twenty CRM?

Twenty is een open-source CRM (Customer Relationship Management) dat je zelf host op een eigen server. Je hebt volledige controle over je klantdata, en het biedt een moderne interface vergelijkbaar met commerciele alternatieven zoals Salesforce of HubSpot -- maar dan zonder maandelijkse licentiekosten per gebruiker.

Twenty heeft standaard ondersteuning voor bedrijven, contactpersonen, deals en taken. Daarnaast kun je eigen objecten en velden aanmaken, en alles besturen via een REST of GraphQL API.

## Wat doet deze skill?

Deze skill begeleidt je -- samen met Claude (of een andere coding agent) -- door het opzetten van een werkende Twenty CRM installatie op een Hetzner VPS met Docker Compose.

Claude doet vooraf onderzoek naar de actuele versie, begeleidt je door de keuzes, en voert de stappen samen met jou uit.

**Geen diepgaande serverkennis vereist** -- wel handig als je weet wat een terminal is.

---

## Hoe begin je?

Zeg dit tegen Claude:

> "Ik wil Twenty CRM installeren op mijn server"

Claude start dan automatisch en loopt met je mee door alles heen -- stap voor stap. Wat Claude zelf kan controleren of regelen, doet Claude. Alleen als iets echt van jou nodig is (zoals een DNS-record aanmaken of een wachtwoord kiezen), vraagt Claude je te stoppen en legt uit wat je moet doen.

---

## Welke combinatie wil je installeren?

Bij de start van de installatie doorloop je een keuzemenu. Je kiest uit:

### Combinatie A -- Alleen Twenty CRM
Twenty CRM met PostgreSQL en Redis op een bestaande of nieuwe VPS. Reverse proxy met nginx of Caddy, SSL via Let's Encrypt.

**Geschikt voor:** Een standalone CRM installatie.

### Combinatie B -- Twenty CRM + n8n
Alles van A, plus n8n als workflow automation platform. Hiermee kun je processen automatiseren: nieuwe leads automatisch in je CRM, e-mail notificaties, koppelingen met andere tools.

**Geschikt voor:** Bedrijven die hun CRM willen integreren met andere systemen.

### Combinatie C -- Twenty CRM + n8n + Custom Objects
Alles van B, plus een op maat ingericht datamodel met eigen objecten, velden en relaties. Denk aan: Projecten, Directories, aangepaste pipelines.

**Geschikt voor:** Consultancy, agencies en dienstverleners die meer nodig hebben dan standaard CRM-objecten.

### Tailscale (optie bij elke combinatie)
Tailscale is een VPN-dienst die zorgt dat de admin-interfaces alleen bereikbaar zijn via een beveiligde prive-verbinding. Webhook endpoints blijven publiek bereikbaar.

- **Met Tailscale:** CRM en n8n alleen via VPN. Veiliger.
- **Zonder Tailscale:** Publiek bereikbaar met login. Extra security headers worden toegepast.

---

## Wat zit er in deze skill?

```
TwentyCRM/
├── SKILL.md              <- Claude's entry point (niet voor jou bedoeld)
├── README.md             <- Dit bestand
├── Research.md           <- Onderzoeksstap (Claude voert dit uit)
├── Workflows/
│   ├── Install.md        <- Installatie-workflow
│   └── Maintain.md       <- Onderhoud-workflow
└── Runbooks/
    ├── 01-Vps.md         <- Stap 1: VPS voorbereiden
    ├── 02-TwentyCRM.md   <- Stap 2: Twenty CRM installeren
    ├── 03-ReverseProxy.md<- Stap 3: Nginx/Caddy + SSL
    ├── 04-n8n.md         <- Stap 4: n8n installeren (optioneel)
    ├── 05-CustomObjects.md <- Stap 5: Datamodel inrichten (optioneel)
    ├── 06-Security.md    <- Stap 6: Security hardening
    ├── 07-Backup.md      <- Stap 7: Backup procedure
    └── 08-Verify.md      <- Stap 8: Alles controleren
```

---

## Technische details

| Component | Versie | Poort |
|-----------|--------|-------|
| Twenty CRM (server + worker) | v1.7.7+ | 3000 |
| PostgreSQL | 16 | 5432 (intern) |
| Redis | 8.2 | 6379 (intern) |
| n8n (optioneel) | latest | 5678 |
| Reverse proxy | nginx of Caddy | 80/443 |

Alle services draaien in Docker containers op `127.0.0.1`. Alleen poort 80 en 443 zijn publiek bereikbaar via de reverse proxy.

---

## Vragen of problemen?

Alle bronnen zijn verifieerbaar:
- Twenty CRM: https://github.com/twentyhq/twenty
- Twenty Docs: https://docs.twenty.com
- n8n: https://github.com/n8n-io/n8n
