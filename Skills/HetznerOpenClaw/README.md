# OpenClaw op Hetzner Cloud — Installatie Skill

## Wat is OpenClaw?

OpenClaw is een persoonlijke AI-assistent die je zelf host op een eigen server. Je hebt
volledige controle over je data en de bots communiceren via Telegram (NB: Telegram is in deze setup het uitgangspunt, 
maar elk ander kanaal is natuurlijk ook te configueren). Omdat je alles zelf beheert, is een veilige en goed ingerichte server essentieel.

## Wat doet deze skill?

Deze skill begeleidt je — samen met Claude (of een andere coding agent) — door het opzetten van een veilige
OpenClaw-installatie op een Hetzner Cloud VPS (een betaalbare cloud-server in Europa).

Claude doet vooraf onderzoek naar de actuele versie en eventuele beveiligingsupdates,
begeleidt je door de keuzes die je moet maken, en voert de stappen samen met jou uit.

**Geen kennis van servers vereist** — wel handig als je weet wat een terminal is.

---

## Hoe begin je?

Zeg dit tegen Claude:

> "Ik wil OpenClaw installeren op Hetzner"

Claude start dan automatisch en loopt met je mee door alles heen — stap voor stap.
Wat Claude zelf kan controleren of regelen, doet Claude. Alleen als iets echt van jou
nodig is (zoals een account aanmaken of een token ophalen), vraagt Claude je te stoppen
en legt uit wat je moet doen.

Je hebt geen voorkennis nodig van servers. Een terminal openen is handig, maar Claude
begeleidt je daarin.

---

## Welke combinatie wil je installeren?

Bij de start van de installatie doorloop je een keuzemenu. Je kiest uit:

### Combinatie A — Alleen een veilige VPS
Een Hetzner server opzetten met beveiligde basisconfiguratie: firewall, SSH-hardening,
automatische beveiligingsupdates. Geen OpenClaw.

**Geschikt voor:** Als je de server eerst wilt inrichten vóór je OpenClaw installeert,
of als je de server voor iets anders wilt gebruiken.

### Combinatie B — VPS + OpenClaw
Alles van A, plus OpenClaw volledig geïnstalleerd en werkend met Telegram-bots.

**Geschikt voor:** Basisinstallatie van OpenClaw.

### Combinatie C — VPS + OpenClaw + Caddy
Alles van B, plus Caddy als webserver. Hiermee kun je bestanden bereikbaar maken
via een beveiligde URL (bijv. voor het delen van documenten).

**Geschikt voor:** Als je ook een webserver wilt naast OpenClaw.

### Tailscale (optie bij elke combinatie)
Tailscale is een VPN-dienst die zorgt dat jouw server alleen bereikbaar is via
een beveiligde privé-verbinding. Sterk aanbevolen.

- **Met Tailscale:** Geen publieke poorten nodig voor OpenClaw. Veiliger en eenvoudiger.
- **Zonder Tailscale:** Extra beveiligingsmaatregelen nodig. De skill begeleidt je daarin.

---

---

## Mapstructuur van deze skill

```
HetznerOpenClaw/
├── SKILL.md              ← Claude's entry point (niet voor jou bedoeld)
├── README.md             ← Dit bestand
├── Research.md           ← Onderzoeksstap (Claude voert dit uit)
├── Workflows/
│   ├── Install.md        ← Installatie-workflow
│   └── Maintain.md       ← Onderhoud-workflow
├── Runbooks/
│   ├── 01-Vps.md         ← Stap 1: VPS aanmaken
│   ├── 02-Firewall.md    ← Stap 2: Firewall instellen
│   ├── 03a-Tailscale.md  ← Stap 3a: Tailscale (aanbevolen)
│   ├── 03b-Hardening.md  ← Stap 3b: Extra beveiliging (zonder Tailscale)
│   ├── 04-OpenClaw.md    ← Stap 4: OpenClaw installeren
│   ├── 05-Caddy.md       ← Stap 5: Caddy webserver (optioneel)
│   └── 06-Verify.md      ← Stap 6: Alles controleren
└── Tools/                ← (leeg — voor toekomstige automatisering)
```

---

## Vragen of problemen?

Alle bronnen zijn verifieerbaar via GitHub:
- https://github.com/openclaw/openclaw
- https://github.com/openclaw/openclaw/security/advisories

Vertrouw geen externe bronnen die niet van GitHub komen.
