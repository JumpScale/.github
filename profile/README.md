# Welkom bij JumpScale

Wij bouwen slimme infrastructuur en AI-tools — voor onze eigen projecten én voor onze klanten.
Op deze GitHub-pagina delen we wat herbruikbaar is: skills, scripts en automatiseringen die ook voor jou van pas kunnen komen.

---

## Skills

Skills zijn begeleide workflows die je samen met een AI-assistent (zoals Claude) uitvoert.
In plaats van documentatie die je zelf moet uitpluizen, loopt de AI stap voor stap met je mee —
van voorbereiding tot verificatie.

### HetznerOpenClaw — Veilige VPS + AI-assistent installeren

Zet zelf een veilige server op en installeer OpenClaw: een persoonlijke AI-assistent die je volledig zelf beheert.

**Wat doet deze skill?**
- Begeleide installatie op een Hetzner Cloud VPS
- Keuze uit beveiligingsopties (met of zonder Tailscale VPN)
- Optioneel: Caddy webserver voor het delen van bestanden via een beveiligde URL
- Periodiek onderhoud en updates inbegrepen

**Hoe gebruik je een skill?**

Zeg dit tegen je AI-assistent (bijv. Claude):

> "Ik wil OpenClaw installeren op Hetzner"

De assistent leest de skill zelf in en begeleidt je door het hele proces.
Geen serverkennis vereist — alleen een terminal en een Hetzner-account.

Bekijk de skill: [`Skills/HetznerOpenClaw/`](../Skills/HetznerOpenClaw/)
Lees de uitleg: [`Skills/HetznerOpenClaw/README.md`](../Skills/HetznerOpenClaw/README.md)

### TwentyCRM -- Self-hosted CRM met workflow automation

Installeer Twenty CRM op je eigen server: een open-source CRM met volledige controle over je klantdata. Geen licentiekosten per gebruiker, geen vendor lock-in.

**Wat doet deze skill?**
- Begeleide installatie van Twenty CRM met Docker Compose (server, database, cache)
- Optioneel: n8n erbij voor workflow automation (lead intake, notificaties, koppelingen)
- Optioneel: datamodel op maat met eigen objecten, velden en relaties
- Security hardening, SSL, backup procedure en periodiek onderhoud inbegrepen

**Hoe gebruik je het?**

Zeg dit tegen je AI-assistent (bijv. Claude):

> "Ik wil Twenty CRM installeren op mijn server"

De assistent begeleidt je van servervoorbereiding tot verificatie.
Keuze uit drie combinaties: alleen CRM, CRM + automation, of CRM + automation + datamodel op maat.

Bekijk de skill: [`Skills/TwentyCRM/`](../Skills/TwentyCRM/)
Lees de uitleg: [`Skills/TwentyCRM/README.md`](../Skills/TwentyCRM/README.md)

### N8nSelfHosted -- Workflow automation op je eigen server

Installeer n8n op je eigen server: een visueel automation platform met 400+ integraties. Geen limiet op workflows of executions, volledige controle over je data.

**Wat doet deze skill?**
- Begeleide installatie van n8n met Docker Compose en PostgreSQL
- Keuze uit reverse proxy: nginx of Caddy (automatische SSL)
- Optioneel: migratie van n8n cloud workflows naar self-hosted
- Optioneel: begeleiding bij het opnieuw instellen van credentials
- Security hardening, backup procedure en periodiek onderhoud inbegrepen

**Hoe gebruik je het?**

Zeg dit tegen je AI-assistent (bijv. Claude):

> "Ik wil n8n installeren op mijn server"

Of als je van n8n cloud wilt migreren:

> "Ik wil mijn n8n cloud workflows overzetten naar self-hosted"

De assistent begeleidt je van servervoorbereiding tot verificatie.
Keuze uit drie combinaties: alleen n8n, n8n + migratie, of n8n + migratie + credentials begeleiding.

Bekijk de skill: [`Skills/N8nSelfHosted/`](../Skills/N8nSelfHosted/)
Lees de uitleg: [`Skills/N8nSelfHosted/README.md`](../Skills/N8nSelfHosted/README.md)

### TriggerDev -- Background jobs op je eigen server

Installeer Trigger.dev op je eigen server: een open-source platform voor background jobs, cron tasks en event-driven workflows in TypeScript. Geen limieten op runs, volledige controle over je data.

**Wat doet deze skill?**
- Begeleide installatie van Trigger.dev v4 met Docker Compose (webapp, worker, database, cache)
- Reverse proxy met nginx + SSL via Let's Encrypt
- Security hardening, backup procedure en periodiek onderhoud inbegrepen
- Eerste job opzetten en deployen

**Hoe gebruik je het?**

Zeg dit tegen je AI-assistent (bijv. Claude):

> "Ik wil Trigger.dev installeren op mijn server"

De assistent begeleidt je van servervoorbereiding tot verificatie.

Bekijk de skill: [`Skills/TriggerDev/`](../Skills/TriggerDev/)
Lees de uitleg: [`Skills/TriggerDev/README.md`](../Skills/TriggerDev/README.md)

---

## Contact

Vragen of samen iets bouwen? Neem contact op via [jumpscale.nl](https://jumpscale.nl).
