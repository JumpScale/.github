# Research  -  Actuele bronnen ophalen

De AI-assistent voert deze stap uit aan het begin van elke Install- of Maintain-sessie.
Doel: zeker weten dat de installatie gebaseerd is op de meest actuele informatie.

## Stap 1: hcloud CLI versie checken

```bash
hcloud version
```

Noteer: versienummer. Als de CLI sterk verouderd is, gebruiker adviseren te updaten
(`brew upgrade hcloud` op Mac).

## Stap 2: Beschikbare server-types controleren

```bash
hcloud server-type list
```

Hetzner hernoemt server-types periodiek. Controleer of het aanbevolen type
(momenteel `cx33` of equivalent) nog beschikbaar is.

## Stap 3: Tailscale installatiewijze controleren

Fetch: https://tailscale.com/download/linux

Noteer of de installatiemethode nog actueel is (curl pipe naar sh).

## Stap 4: Bevindingen samenvatten

Na het ophalen van bovenstaande informatie, geef de gebruiker een korte samenvatting:

```
Research-uitkomst:
- hcloud CLI: [versie]
- Aanbevolen server-type: [type] ([specs])
- Tailscale installatie: [methode]
- Aandachtspunten: [eventuele bijzonderheden]
```

## Bronnen-protocol

**Alleen deze bronnen zijn toegestaan:**

| Bron | URL |
|------|-----|
| Hetzner API docs | https://docs.hetzner.cloud/reference/cloud |
| Tailscale docs | https://tailscale.com/kb |
| Ubuntu security | https://ubuntu.com/security/notices |

**NOOIT:** willekeurige blogs, LinkedIn, Substack, AI-gegenereerde artikelen.
