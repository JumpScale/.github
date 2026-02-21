# Trigger.dev Self-Hosted -- Installatie Skill

## Wat is Trigger.dev?

Trigger.dev is een background job processing platform voor TypeScript/JavaScript. Denk aan: langlopende taken, geplande jobs, event-driven workflows, en alles wat je niet in een request/response cyclus wilt afhandelen. Trigger.dev biedt een visuele dashboard, retry-logica, en integratie met je bestaande TypeScript codebase.

Door Trigger.dev zelf te hosten heb je volledige controle over je data, geen limiet op het aantal runs, en geen maandelijkse kosten per task execution.

## Wat doet deze skill?

Deze skill begeleidt je -- samen met Claude (of een andere coding agent) -- door het opzetten van een werkende Trigger.dev installatie op een eigen server. Inclusief Docker Compose configuratie, reverse proxy, SSL, en security hardening.

Claude doet vooraf onderzoek naar de actuele versie, begeleidt je door de keuzes, en voert de stappen samen met jou uit.

**Geen diepgaande serverkennis vereist** -- wel handig als je weet wat een terminal is.

---

## Hoe begin je?

Zeg dit tegen Claude:

> "Ik wil Trigger.dev installeren op mijn server"

Of voor onderhoud:

> "Voer onderhoud uit op mijn Trigger.dev server"

Claude start dan automatisch en loopt met je mee door alles heen -- stap voor stap.

---

## Welke combinatie wil je installeren?

### Combinatie A -- Alleen Trigger.dev
Trigger.dev met Docker Compose op een bestaande of nieuwe VPS. Reverse proxy met nginx, SSL via Let's Encrypt.

**Geschikt voor:** Een fresh start met self-hosted background job processing.

### Combinatie B -- Trigger.dev + project setup
Alles van A, plus begeleiding bij het configureren van de SDK in je TypeScript project en het deployen van je eerste task.

**Geschikt voor:** Direct productief aan de slag met Trigger.dev.

### Tailscale (optie bij elke combinatie)
Het Trigger.dev dashboard alleen bereikbaar via VPN. API endpoints blijven publiek zodat je applicatie er bij kan.

- **Met Tailscale:** Dashboard alleen via prive-netwerk. Veiliger.
- **Zonder Tailscale:** Publiek bereikbaar met login + security headers.

---

## Architectuur

Trigger.dev self-hosted draait als een set Docker containers:

| Service | Functie |
|---------|---------|
| **webapp** | Dashboard + API (poort 3040 extern, 3030 intern) |
| **postgres** | Database |
| **redis** | Queue en caching |
| **docker-provider** | Beheert task execution containers |
| **coordinator** | Coordineert task scheduling |
| **electric** | Real-time data sync |

Authenticatie gaat via magic link: je voert je e-mailadres in, en de login-link verschijnt in de container logs.

---

## Mapstructuur van deze skill

```
TriggerDev/
├── SKILL.md              <- Claude's entry point (niet voor jou bedoeld)
├── README.md             <- Dit bestand
├── Research.md           <- Onderzoeksstap (Claude voert dit uit)
├── Workflows/
│   ├── Install.md        <- Installatie-workflow
│   └── Maintain.md       <- Onderhoud-workflow
└── Runbooks/
    ├── 01-Vps.md         <- Stap 1: Server voorbereiden
    ├── 02-TriggerDev.md  <- Stap 2: Trigger.dev installeren
    ├── 03-ReverseProxy.md<- Stap 3: Nginx + SSL
    ├── 04-Security.md    <- Stap 4: Security hardening
    ├── 05-Backup.md      <- Stap 5: Backup procedure
    └── 06-Verify.md      <- Stap 6: Alles controleren
```

---

## Vragen of problemen?

Alle bronnen zijn verifieerbaar:
- https://github.com/triggerdotdev/trigger.dev
- https://trigger.dev/docs
- https://github.com/triggerdotdev/docker
