# OpenClaw  -  Persoonlijke AI-Assistent

## Wat is OpenClaw?

OpenClaw is een persoonlijke AI-assistent die je zelf host op een eigen server. Je hebt
volledige controle over je data en de bots communiceren via Telegram. Omdat je alles
zelf beheert, is je data van jou  -  niet van een derde partij.

## Wat doet deze skill?

Deze skill begeleidt je door het opzetten van een werkende OpenClaw-installatie op
je eigen server. De AI-assistent doet vooraf onderzoek naar de actuele versie en
eventuele beveiligingsupdates, en voert de stappen samen met jou uit.

**Geen serverkennis vereist**  -  wel een werkende VPS met SSH-toegang.

> Heb je nog geen server? Gebruik eerst de **[HetznerVPS](../HetznerVPS/)** skill
> om een veilige VPS op te zetten.

---

## Hoe begin je?

Zeg dit tegen je AI-assistent:

> "Ik wil OpenClaw installeren op mijn server"

De assistent start automatisch en loopt met je mee  -  stap voor stap.

---

## Wat krijg je?

### Altijd inbegrepen
- **OpenClaw gateway** als Docker container
- **Telegram-bot** verbonden en werkend
- **Automatische backups** (dagelijkse SQLite backup + configuratie)
- **Rollback-tag** bij elke update  -  terugdraaien is altijd mogelijk

### Optioneel: Caddy webserver
Caddy is een webserver die bestanden beschikbaar maakt via een beveiligde URL.
Handig voor het delen van documenten, exports of rapporten die OpenClaw aanmaakt.

---

## Vereisten

- Een werkende VPS met SSH-toegang
- Docker geïnstalleerd op de VPS
- Telegram Bot Token (aan te maken via @BotFather  -  de assistent helpt je daarmee)
- Anthropic API-sleutel (voor AI-functionaliteit)

---

## Na installatie

Gebruik de **Maintain**-workflow voor onderhoud en updates:

> "Voer onderhoud uit op mijn OpenClaw"

Dit controleert de status, maakt backups, en updatet naar de nieuwste versie als die beschikbaar is.

---

## Mapstructuur

```
OpenClaw/
├── SKILL.md              ← Entry point voor de AI-assistent
├── README.md             ← Dit bestand
├── Research.md           ← Onderzoeksstap (AI voert dit uit)
├── Workflows/
│   ├── Install.md        ← Installatie-workflow
│   └── Maintain.md       ← Onderhoud-workflow
└── Runbooks/
    ├── 01-Docker.md      ← Docker installeren (indien nodig)
    ├── 02-OpenClaw.md    ← OpenClaw installeren en configureren
    ├── 03-Caddy.md       ← Caddy webserver (optioneel)
    └── 04-Verify.md      ← Verificatie-checklist
```
