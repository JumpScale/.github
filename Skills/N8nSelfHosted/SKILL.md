---
name: N8nSelfHosted
description: Begeleide installatie en onderhoud van n8n workflow automation op een eigen server met Docker Compose. USE WHEN n8n installeren, n8n opzetten, n8n deployen, self-hosted n8n, workflow automation, n8n server, n8n onderhoud, n8n backup, n8n migratie, n8n cloud naar self-hosted, webhook server, automation platform.
---

# N8nSelfHosted

Begeleide installatie en onderhoud van n8n op een eigen server.
Geschikt voor technisch onderlegde gebruikers die workflow automation willen zonder afhankelijkheid van n8n cloud.

## Workflow Routing

**When executing a workflow, output this notification:**

```
Running the **WorkflowName** workflow from the **N8nSelfHosted** skill...
```

| Workflow | Trigger | File |
|----------|---------|------|
| **Install** | "installeer n8n", "n8n opzetten", "n8n deployen", "self-hosted n8n", "workflow automation server" | `Workflows/Install.md` |
| **Migrate** | "n8n migreren", "cloud naar self-hosted", "workflows overzetten", "n8n overzetten" | `Workflows/Migrate.md` |
| **Maintain** | "onderhoud n8n", "update n8n", "backup n8n", "n8n controleren" | `Workflows/Maintain.md` |

## Hoe werkt dit?

1. Je kiest **Install**, **Migrate** of **Maintain**
2. Claude voert eerst een **Research-stap** uit -- actuele n8n versie, changelogs en security-advisories worden opgehaald
3. Bij Install doorloop je een **keuzemenu**: welke componenten wil je installeren?
4. Bij Migrate begeleidt Claude je door het exporteren en importeren van workflows
5. Claude begeleidt je stap voor stap door de relevante runbooks
6. Aan het einde loop je een **verificatie-checklist** na

## Combinaties die je kunt kiezen

| Combinatie | Inhoud |
|-----------|--------|
| A | n8n standalone (Docker, reverse proxy, SSL) |
| B | A + migratie van n8n cloud workflows |
| C | A + migratie + externe koppelingen (CRM, Airtable, etc.) |
| + Tailscale | Toe te voegen aan elke combinatie (aanbevolen) |

## Bronnen (alleen verifieerbaar)

- n8n GitHub: https://github.com/n8n-io/n8n
- n8n Docs: https://docs.n8n.io
- n8n Releases: https://github.com/n8n-io/n8n/releases
- n8n Docker Hub: https://hub.docker.com/r/n8nio/n8n

**Nooit gebruiken:** willekeurige blogs, LinkedIn-artikelen, Substack, niet-officiele tutorials.

## Examples

**Example 1: Fresh install**
```
User: "Ik wil n8n installeren op mijn server"
-> Invokes Install workflow
-> Research: actuele versie ophalen
-> Wizard: keuze combinatie (A/B/C) + Tailscale ja/nee
-> Runbooks in volgorde uitvoeren
-> Verificatie-checklist nalopen
```

**Example 2: Migratie van cloud**
```
User: "Ik wil mijn n8n cloud workflows overzetten naar self-hosted"
-> Invokes Migrate workflow
-> Export workflows via cloud API
-> Opschonen van foreign key referenties
-> Import via n8n CLI
-> Credentials opnieuw instellen
```

**Example 3: Periodiek onderhoud**
```
User: "Voer onderhoud uit op mijn n8n server"
-> Invokes Maintain workflow
-> Research: nieuwe versie beschikbaar?
-> Checks: container, disk, backups, SSL, workflows
-> Update als nieuwe versie beschikbaar
```
