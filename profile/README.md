<div align="center">

# JumpScale

**AI-begeleide installatiescripts voor je eigen infrastructuur**

*Geen handboeken doorploegen. Zeg wat je wilt  -  je AI-assistent loopt mee.*

</div>

---

## Hoe werkt dit?

Onze **Skills** zijn instructiesets die een AI-assistent (zoals [Claude Code](https://docs.anthropic.com/en/docs/claude-code)) stap voor stap door technische processen leidt. De assistent doet onderzoek, voert commando's uit, legt uit wat er gebeurt, en controleert of alles werkt.

Jij hebt alleen nodig:
- Een AI coding agent (bijv. Claude Code)
- Een terminal
- Een account bij de relevante provider

**Geen serverkennis vereist.**

---

## Skills

| Skill | Wat het doet | Zeg dit tegen je assistent |
|-------|-------------|---------------------------|
| **[HetznerVPS](Skills/HetznerVPS/)** | Veilige VPS opzetten op Hetzner Cloud  -  firewall, SSH-hardening, Tailscale | *"Ik wil een veilige VPS opzetten op Hetzner"* |
| **[OpenClaw](Skills/OpenClaw/)** | Persoonlijke AI-assistent installeren op je eigen server | *"Ik wil OpenClaw installeren op mijn server"* |
| **[TwentyCRM](Skills/TwentyCRM/)** | Open-source CRM met optionele workflow automation via n8n | *"Ik wil Twenty CRM installeren op mijn server"* |
| **[N8nSelfHosted](Skills/N8nSelfHosted/)** | Visueel automation platform met 400+ integraties | *"Ik wil n8n installeren op mijn server"* |
| **[TriggerDev](Skills/TriggerDev/)** | Background jobs en cron tasks in TypeScript | *"Ik wil Trigger.dev installeren op mijn server"* |

> Elke skill bevat zowel installatie als onderhoud. Security hardening en backups zitten standaard inbegrepen.

---

## Voorbeeld

```
Jij:     "Ik wil een veilige VPS opzetten op Hetzner"

Claude:  → Checkt of hcloud CLI geinstalleerd is
         → Vraagt welke onderdelen je wilt (Tailscale? Caddy?)
         → Maakt de server aan en configureert firewall
         → Loopt een verificatie-checklist na
         → Klaar. Server draait.
```

---

## Aan de slag

1. Clone deze repo
2. Open de repo in je AI coding agent
3. Zeg wat je wilt installeren

De assistent leest de skill automatisch in en begeleidt je door het hele proces.

---

<div align="center">

Vragen of feedback? Neem contact op via [jumpscale.nl](https://jumpscale.nl)

</div>
