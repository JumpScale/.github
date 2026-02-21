---
name: TwentyCRM
description: Begeleide installatie en onderhoud van Twenty CRM op een eigen server met Docker Compose. USE WHEN twenty crm installeren, crm opzetten, twenty deployen, self-hosted crm, klantbeheer systeem, crm server, twenty onderhoud, twenty backup, crm datamodel, custom objects twenty, n8n crm koppeling, crm automatisering.
---

# TwentyCRM

Begeleide installatie en onderhoud van Twenty CRM op een eigen server.
Geschikt voor technisch onderlegde gebruikers die een professioneel CRM willen zonder vendor lock-in.

## Workflow Routing

**When executing a workflow, output this notification:**

```
Running the **WorkflowName** workflow from the **TwentyCRM** skill...
```

| Workflow | Trigger | File |
|----------|---------|------|
| **Install** | "installeer twenty crm", "crm opzetten", "twenty deployen", "self-hosted crm" | `Workflows/Install.md` |
| **Maintain** | "onderhoud crm", "update twenty", "backup crm", "crm controleren" | `Workflows/Maintain.md` |

## Hoe werkt dit?

1. Je kiest **Install** of **Maintain**
2. Claude voert eerst een **Research-stap** uit -- actuele Twenty versie, changelogs en security-advisories worden opgehaald
3. Bij Install doorloop je een **keuzemenu**: welke componenten wil je installeren?
4. Claude begeleidt je stap voor stap door de relevante runbooks
5. Aan het einde loop je een **verificatie-checklist** na

## Combinaties die je kunt kiezen

| Combinatie | Inhoud |
|-----------|--------|
| A | Twenty CRM (server, worker, postgres, redis) |
| B | A + n8n (workflow automation) |
| C | A + n8n + Custom Objects (datamodel op maat) |
| + Tailscale | Toe te voegen aan elke combinatie (aanbevolen) |

## Bronnen (alleen verifieerbaar)

- Twenty CRM: https://github.com/twentyhq/twenty
- Twenty Docs: https://docs.twenty.com
- Twenty Releases: https://github.com/twentyhq/twenty/releases
- n8n: https://github.com/n8n-io/n8n
- Hetzner docs: https://docs.hetzner.cloud/reference/cloud

**Nooit gebruiken:** willekeurige blogs, LinkedIn-artikelen, Substack, niet-officiele tutorials.

## Examples

**Example 1: Volledige installatie met n8n**
```
User: "Ik wil Twenty CRM installeren op mijn Hetzner server"
-> Invokes Install workflow
-> Research: actuele versie + security fixes ophalen
-> Wizard: keuze combinatie (A/B/C) + Tailscale ja/nee
-> Runbooks in volgorde uitvoeren
-> Verificatie-checklist nalopen
```

**Example 2: Periodiek onderhoud**
```
User: "Voer onderhoud uit op mijn Twenty CRM"
-> Invokes Maintain workflow
-> Research: nieuwe versie beschikbaar? security advisories?
-> Checks: containers, database, backups, SSL, disk
-> Update-procedure als nieuwe versie beschikbaar
```

**Example 3: Datamodel uitbreiden**
```
User: "Ik wil een Project-object toevoegen aan mijn CRM"
-> Research: huidige objecten en relaties ophalen via API
-> Custom object aanmaken via REST metadata API
-> Velden en relaties configureren
-> Verificatie in Twenty UI
```
