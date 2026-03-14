---
name: OpenClaw
description: Installatie en onderhoud van OpenClaw op een eigen server. USE WHEN openclaw installeren, openclaw deployen, openclaw updaten, ai assistent installeren, telegram bot installeren, openclaw onderhoud, caddy webserver, openclaw backup.
---

# OpenClaw

Begeleide installatie en onderhoud van OpenClaw — een persoonlijke AI-assistent
die je zelf host op je eigen server.

## Workflow Routing

**When executing a workflow, output this notification:**

```
Running the **WorkflowName** workflow from the **OpenClaw** skill...
```

| Workflow | Trigger | File |
|----------|---------|------|
| **Install** | "installeer openclaw", "openclaw opzetten", "ai assistent installeren" | `Workflows/Install.md` |
| **Maintain** | "openclaw onderhoud", "openclaw update", "openclaw backup", "openclaw logs" | `Workflows/Maintain.md` |

## Vereisten

- Een werkende VPS met SSH-toegang (zie de **[HetznerVPS](../HetznerVPS/)** skill als je nog geen server hebt)
- Docker geïnstalleerd op de server
- Telegram Bot Token (via @BotFather)

## Wat levert dit op?

- OpenClaw gateway draaiend als Docker container
- Telegram-bot verbonden en werkend
- Optioneel: Caddy webserver voor het delen van bestanden
- Automatische backups (dagelijks)
- Rollback-mogelijkheid bij updates

## Combinaties

| Keuze | Inhoud |
|-------|--------|
| Basis | OpenClaw gateway + Telegram-bot |
| + Caddy | Webserver erbij voor het delen van bestanden via een URL |

## Bronnen (alleen verifieerbaar)

- Codebase: https://github.com/openclaw/openclaw
- Changelog: https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md
- Security advisories: https://github.com/openclaw/openclaw/security/advisories

**Nooit gebruiken:** openclaw.report, willekeurige blogs, LinkedIn-artikelen, Substack.

## Examples

**Example 1: Nieuwe installatie**
```
User: "Ik wil OpenClaw installeren op mijn server"
-> Invokes Install workflow
-> Research: actuele versie + security fixes ophalen
-> Vereisten checken (Docker, Telegram token)
-> OpenClaw installeren en configureren
-> Optioneel: Caddy webserver installeren
-> Verificatie-checklist nalopen
```

**Example 2: Update uitvoeren**
```
User: "Update OpenClaw naar de laatste versie"
-> Invokes Maintain workflow
-> Research: nieuwe versie beschikbaar? security advisories?
-> Backup aanmaken
-> Image bouwen en deployen
-> Verificatie na update
```
