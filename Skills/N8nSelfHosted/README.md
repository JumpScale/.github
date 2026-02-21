# n8n Self-Hosted -- Installatie Skill

## Wat is n8n?

n8n is een workflow automation platform waarmee je processen automatiseert zonder code te schrijven. Denk aan: nieuwe lead binnenkomt via formulier, automatisch in je CRM, e-mail notificatie naar sales, Slack bericht naar het team. n8n heeft 400+ integraties en een visuele editor.

Door n8n zelf te hosten heb je volledige controle over je data, geen limiet op het aantal workflows, en geen maandelijkse kosten per execution.

## Wat doet deze skill?

Deze skill begeleidt je -- samen met Claude (of een andere coding agent) -- door het opzetten van een werkende n8n installatie op een eigen server. Inclusief migratie van bestaande workflows als je van n8n cloud overstapt.

Claude doet vooraf onderzoek naar de actuele versie, begeleidt je door de keuzes, en voert de stappen samen met jou uit.

**Geen diepgaande serverkennis vereist** -- wel handig als je weet wat een terminal is.

---

## Hoe begin je?

Zeg dit tegen Claude:

> "Ik wil n8n installeren op mijn server"

Of als je van n8n cloud wilt migreren:

> "Ik wil mijn n8n cloud workflows overzetten naar self-hosted"

Claude start dan automatisch en loopt met je mee door alles heen -- stap voor stap.

---

## Welke combinatie wil je installeren?

### Combinatie A -- Alleen n8n
n8n met Docker Compose op een bestaande of nieuwe VPS. Reverse proxy met nginx of Caddy, SSL via Let's Encrypt.

**Geschikt voor:** Een fresh start met self-hosted automation.

### Combinatie B -- n8n + migratie van cloud
Alles van A, plus automatische migratie van al je workflows vanuit n8n cloud. Workflows worden geexporteerd via de cloud API, opgeschoond, en geimporteerd in je self-hosted instance.

**Geschikt voor:** Overstappen van n8n cloud naar self-hosted.

### Combinatie C -- n8n + migratie + koppelingen
Alles van B, plus begeleiding bij het opnieuw instellen van credentials en het koppelen aan externe services (CRM, Airtable, Google, OpenAI, etc.).

**Geschikt voor:** Volledige migratie inclusief alle integraties.

### Tailscale (optie bij elke combinatie)
De n8n admin-interface alleen bereikbaar via VPN. Webhook URLs blijven publiek zodat externe services (formulieren, apps) er nog bij kunnen.

- **Met Tailscale:** n8n editor alleen via prive-netwerk. Veiliger.
- **Zonder Tailscale:** Publiek bereikbaar met login + security headers.

---

## Belangrijk: credentials migreren niet automatisch

n8n versleutelt credentials (API keys, OAuth tokens). Bij migratie van cloud naar self-hosted worden workflows wel overgezet, maar credentials **niet**. Die moet je opnieuw instellen in de self-hosted n8n UI.

Deze skill begeleidt je daarbij: per workflow wordt aangegeven welke credentials nodig zijn.

---

## Mapstructuur van deze skill

```
N8nSelfHosted/
├── SKILL.md              <- Claude's entry point (niet voor jou bedoeld)
├── README.md             <- Dit bestand
├── Research.md           <- Onderzoeksstap (Claude voert dit uit)
├── Workflows/
│   ├── Install.md        <- Installatie-workflow
│   ├── Migrate.md        <- Migratie-workflow (cloud naar self-hosted)
│   └── Maintain.md       <- Onderhoud-workflow
└── Runbooks/
    ├── 01-Vps.md         <- Stap 1: Server voorbereiden
    ├── 02-N8n.md         <- Stap 2: n8n installeren
    ├── 03-ReverseProxy.md<- Stap 3: Nginx/Caddy + SSL
    ├── 04-Migration.md   <- Stap 4: Workflows migreren (optioneel)
    ├── 05-Credentials.md <- Stap 5: Credentials instellen (optioneel)
    ├── 06-Security.md    <- Stap 6: Security hardening
    ├── 07-Backup.md      <- Stap 7: Backup procedure
    └── 08-Verify.md      <- Stap 8: Alles controleren
```

---

## Vragen of problemen?

Alle bronnen zijn verifieerbaar:
- https://github.com/n8n-io/n8n
- https://docs.n8n.io
