---
name: TriggerDev
description: Begeleide installatie en onderhoud van Trigger.dev v4 background job platform op een eigen server met Docker Compose. USE WHEN trigger.dev installeren, trigger.dev opzetten, trigger.dev deployen, self-hosted trigger.dev, background jobs, task queue, trigger.dev server, trigger.dev onderhoud, trigger.dev backup, job processing, typescript background tasks.
---

# TriggerDev

Begeleide installatie en onderhoud van Trigger.dev op een eigen server.
Geschikt voor technisch onderlegde gebruikers die background job processing willen zonder afhankelijkheid van Trigger.dev cloud.

## Workflow Routing

**When executing a workflow, output this notification:**

```
Running the **WorkflowName** workflow from the **TriggerDev** skill...
```

| Workflow | Trigger | File |
|----------|---------|------|
| **Install** | "installeer trigger.dev", "trigger.dev opzetten", "trigger.dev deployen", "self-hosted trigger.dev", "background job server" | `Workflows/Install.md` |
| **Maintain** | "onderhoud trigger.dev", "update trigger.dev", "backup trigger.dev", "trigger.dev controleren" | `Workflows/Maintain.md` |

## Hoe werkt dit?

1. Je kiest **Install** of **Maintain**
2. Claude voert eerst een **Research-stap** uit -- actuele Trigger.dev versie, changelogs en security-advisories worden opgehaald
3. Bij Install doorloop je een **keuzemenu**: welke componenten wil je installeren?
4. Claude begeleidt je stap voor stap door de relevante runbooks
5. Aan het einde loop je een **verificatie-checklist** na

## Combinaties die je kunt kiezen

| Combinatie | Inhoud |
|-----------|--------|
| A | Trigger.dev standalone (Docker Compose, reverse proxy, SSL) |
| B | A + project setup (SDK configuratie, eerste task deployen) |
| + Tailscale | Toe te voegen aan elke combinatie (aanbevolen) |

## Bronnen (alleen verifieerbaar)

- Trigger.dev GitHub: https://github.com/triggerdotdev/trigger.dev
- Trigger.dev Docs: https://trigger.dev/docs
- Trigger.dev Docker: https://github.com/triggerdotdev/docker
- Trigger.dev Releases: https://github.com/triggerdotdev/trigger.dev/releases
- Docker images: ghcr.io/triggerdotdev/trigger.dev, ghcr.io/triggerdotdev/provider/docker, ghcr.io/triggerdotdev/coordinator

**Nooit gebruiken:** willekeurige blogs, LinkedIn-artikelen, Substack, niet-officiele tutorials.

## Examples

**Example 1: Fresh install**
```
User: "Ik wil Trigger.dev installeren op mijn server"
-> Invokes Install workflow
-> Research: actuele versie ophalen
-> Wizard: keuze combinatie (A/B) + Tailscale ja/nee
-> Runbooks in volgorde uitvoeren
-> Verificatie-checklist nalopen
```

**Example 2: Periodiek onderhoud**
```
User: "Voer onderhoud uit op mijn Trigger.dev server"
-> Invokes Maintain workflow
-> Research: nieuwe versie beschikbaar?
-> Checks: containers, disk, backups, SSL
-> Update als nieuwe versie beschikbaar
```
