---
name: HetznerVPS
description: Veilige installatie en hardening van een Hetzner Cloud VPS. USE WHEN hetzner vps opzetten, vps inrichten, server setup, tailscale installeren, server hardening, firewall instellen, veilige server, vps beheer, security check vps.
---

# HetznerVPS

Begeleide installatie en hardening van een Hetzner Cloud VPS.
Levert een productie-klare server op met dual-layer firewall, SSH-hardening en optioneel Tailscale VPN.

## Workflow Routing

**When executing a workflow, output this notification:**

```
Running the **WorkflowName** workflow from the **HetznerVPS** skill...
```

| Workflow | Trigger | File |
|----------|---------|------|
| **Install** | "nieuwe vps", "vps opzetten", "setup hetzner", "server aanmaken" | `Workflows/Install.md` |
| **Maintain** | "vps onderhoud", "security check", "vps controleren", "firewall check" | `Workflows/Maintain.md` |

## Wat levert dit op?

Een Hetzner Cloud VPS met:
- SSH key-only authenticatie (wachtwoord uitgeschakeld)
- Dual-layer firewall (Hetzner Cloud Firewall + UFW)
- Automatische beveiligingsupdates (unattended-upgrades)
- Optioneel: Tailscale VPN voor veilige privétoegang
- Optioneel: extra hardening zonder Tailscale (fail2ban, IP-restrictie)

## Combinaties

| Keuze | Inhoud |
|-------|--------|
| Basis | VPS + firewall + SSH-hardening + automatische updates |
| + Tailscale | Privétoegang via Tailscale mesh VPN (aanbevolen) |
| Zonder Tailscale | Extra hardening: fail2ban, IP-restrictie, publieke HTTPS |

## Na de VPS setup

Na een werkende VPS kun je software installeren met deze skills:

| Skill | Wat | Link |
|-------|-----|------|
| **N8nSelfHosted** | Workflow automation (Docker) | [→ Skill](../N8nSelfHosted/) |
| **TwentyCRM** | CRM systeem (Docker) | [→ Skill](../TwentyCRM/) |
| **OpenClaw** | AI-assistent + Telegram bot | [→ Skill](../OpenClaw/) |
| **TriggerDev** | Background job processing (Docker, 8+ GB RAM) | [→ Skill](../TriggerDev/) |

## Bronnen (alleen verifieerbaar)

- Hetzner docs: https://docs.hetzner.cloud/reference/cloud
- Tailscale docs: https://tailscale.com/kb

**Nooit gebruiken:** willekeurige blogs, LinkedIn-artikelen, Substack.

## Examples

**Example 1: Nieuwe VPS opzetten**
```
User: "Ik wil een veilige VPS opzetten op Hetzner"
-> Invokes Install workflow
-> Vereisten checken (hcloud CLI, SSH key, Hetzner account)
-> VPS aanmaken, firewall configureren
-> Tailscale of hardening instellen
-> Verificatie-checklist nalopen
```

**Example 2: Security check**
```
User: "Controleer de security van mijn VPS"
-> Invokes Maintain workflow
-> Firewall-regels controleren (Hetzner + UFW)
-> SSH config verifiëren
-> Updates checken
-> Disk en geheugen controleren
```
