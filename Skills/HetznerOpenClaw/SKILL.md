---
name: HetznerOpenClaw
description: Veilige installatie en periodiek onderhoud van OpenClaw op Hetzner Cloud VPS. USE WHEN openclaw installeren, hetzner vps opzetten, vps inrichten, openclaw deployen, server setup, tailscale installeren, caddy webserver, server onderhoud, vps beheer, openclaw updaten, security check vps, backup openclaw.
---

# HetznerOpenClaw

Begeleide installatie en onderhoud van OpenClaw op een Hetzner Cloud VPS.
Geschikt voor technisch onderlegde gebruikers die zelf een veilige instantie willen opzetten.

## Workflow Routing

**When executing a workflow, output this notification:**

```
Running the **WorkflowName** workflow from the **HetznerOpenClaw** skill...
```

| Workflow | Trigger | File |
|----------|---------|------|
| **Install** | "installeer openclaw", "nieuwe vps", "openclaw opzetten", "setup hetzner" | `Workflows/Install.md` |
| **Maintain** | "onderhoud", "update openclaw", "backup", "security check", "vps controleren" | `Workflows/Maintain.md` |

## Hoe werkt dit?

1. Je kiest **Install** of **Maintain**
2. Claude voert eerst een **Research-stap** uit — actuele OpenClaw-versie, changelogs en security-advisories worden opgehaald
3. Bij Install doorloop je een **keuzemenu**: welke componenten wil je installeren?
4. Claude begeleid je stap voor stap door de relevante runbooks
5. Aan het einde loop je een **verificatie-checklist** na

## Combinaties die je kunt kiezen

| Combinatie | Inhoud |
|-----------|--------|
| A | Hetzner VPS (secure baseline) |
| B | A + OpenClaw |
| C | A + OpenClaw + Caddy (webserver) |
| + Tailscale | Toe te voegen aan elke combinatie (aanbevolen) |

## Bronnen (alleen verifieerbaar)

- Codebase: https://github.com/openclaw/openclaw
- Changelog: https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md
- Security advisories: https://github.com/openclaw/openclaw/security/advisories
- Hetzner docs: https://docs.hetzner.cloud/reference/cloud

**Nooit gebruiken:** openclaw.report, willekeurige blogs, LinkedIn-artikelen, Substack.

## Examples

**Example 1: Volledige installatie**
```
User: "Ik wil OpenClaw installeren op Hetzner"
-> Invokes Install workflow
-> Research: actuele versie + security fixes ophalen
-> Wizard: keuze combinatie (A/B/C) + Tailscale ja/nee
-> Runbooks in volgorde uitvoeren
-> Verificatie-checklist nalopen
```

**Example 2: Periodiek onderhoud**
```
User: "Voer onderhoud uit op mijn OpenClaw VPS"
-> Invokes Maintain workflow
-> Research: nieuwe versie beschikbaar? security advisories?
-> Checks: containers, firewall, backups, disk, reboot nodig?
-> Update-procedure als nieuwe versie beschikbaar
```
