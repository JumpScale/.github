# Hetzner VPS  -  Veilige Server Setup

## Wat doet deze skill?

Deze skill begeleidt je  -  samen met een AI-assistent  -  door het opzetten van een veilige
Hetzner Cloud VPS. Je krijgt een productie-klare server met SSH-hardening, dual-layer
firewall en optioneel Tailscale VPN.

**Geen serverkennis vereist**  -  de AI begeleidt je stap voor stap.

---

## Hoe begin je?

Zeg dit tegen je AI-assistent:

> "Ik wil een veilige VPS opzetten op Hetzner"

De assistent checkt automatisch of alles klaar is (hcloud CLI, SSH key, Hetzner account),
maakt de server aan, configureert de beveiliging en loopt een verificatie-checklist na.

---

## Wat krijg je?

Elke installatie bevat:

- **Hetzner Cloud VPS** met Ubuntu  -  aangemaakt via de hcloud CLI
- **SSH-hardening**  -  alleen key-based login, wachtwoord uitgeschakeld
- **Dual-layer firewall**  -  Hetzner Cloud Firewall (netwerkniveau) + UFW (OS-niveau)
- **Automatische beveiligingsupdates** via unattended-upgrades

Daarnaast kies je:

### Met Tailscale (aanbevolen)
Tailscale is een VPN-dienst die een privaat netwerk aanmaakt. Je server is alleen
bereikbaar via dit netwerk  -  niet via het publieke internet.

- Geen extra publieke poorten nodig
- Automatisch certificaatbeheer
- Toegang eenvoudig intrekken via de Tailscale console

### Zonder Tailscale
Extra beveiligingsmaatregelen compenseren de publieke blootstelling:

- **fail2ban**  -  automatische IP-blokkering bij herhaalde mislukte pogingen
- **IP-restrictie**  -  optioneel: alleen toegang vanuit bekende IP-adressen
- **HTTPS via Let's Encrypt**  -  verplicht voor elke service die je publiek maakt

---

## Vereisten

- hcloud CLI geïnstalleerd (de assistent helpt je daarmee)
- Hetzner Cloud account
- SSH keypair op je lokale machine
- Terminal-toegang

---

## Na installatie

Gebruik de **Maintain**-workflow voor periodiek onderhoud:

> "Controleer de security van mijn VPS"

Dit controleert firewall-regels, SSH-config, beschikbare updates en disk-gebruik.

---

## Mapstructuur

```
HetznerVPS/
├── SKILL.md              ← Entry point voor de AI-assistent
├── README.md             ← Dit bestand
├── Research.md           ← Onderzoeksstap (AI voert dit uit)
├── Workflows/
│   ├── Install.md        ← Installatie-workflow
│   └── Maintain.md       ← Onderhoud-workflow
└── Runbooks/
    ├── 01-Vps.md         ← VPS aanmaken en basisconfiguratie
    ├── 02-Firewall.md    ← Dual-layer firewall
    ├── 03a-Tailscale.md  ← Tailscale VPN (aanbevolen)
    ├── 03b-Hardening.md  ← Extra hardening zonder Tailscale
    └── 04-Verify.md      ← Verificatie-checklist
```
