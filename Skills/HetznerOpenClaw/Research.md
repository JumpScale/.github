# Research — Actuele bronnen ophalen

Claude voert deze stap uit aan het begin van elke Install- of Maintain-sessie.
Doel: zeker weten dat de installatie gebaseerd is op de meest actuele informatie.

## Stap 1: OpenClaw changelog ophalen

Fetch: https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md

Noteer:
- Meest recente versie-nummer
- Datum van laatste release
- Security fixes in de laatste versie(s)
- Nieuwe poorten, services of dependencies die de installatie beïnvloeden

## Stap 2: Security advisories controleren

Fetch: https://github.com/openclaw/openclaw/security/advisories

Noteer:
- Openstaande of recente CVEs
- Aanbevolen mitigaties
- Of de huidige installatie (indien Maintain) kwetsbaar is

## Stap 3: Hetzner hcloud CLI versie checken

```bash
hcloud version
```

Noteer: versienummer. Als de CLI sterk verouderd is, gebruiker adviseren te updaten
(`brew upgrade hcloud` op Mac).

## Stap 4: Bevindingen samenvatten

Na het ophalen van bovenstaande informatie, geef de gebruiker een korte samenvatting:

```
Research-uitkomst:
- OpenClaw versie: [versie] ([datum])
- Security fixes: [samenvatting of "geen kritieke issues"]
- Aandachtspunten voor deze installatie: [eventuele bijzonderheden]
- hcloud CLI: [versie]
```

Als er kritieke security issues zijn: benoem ze expliciet en vraag of de gebruiker
wil doorgaan met de installatie.

## Bronnen-protocol

**Alleen deze bronnen zijn toegestaan:**

| Bron | URL |
|------|-----|
| OpenClaw GitHub | https://github.com/openclaw/openclaw |
| Changelog | https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md |
| Security advisories | https://github.com/openclaw/openclaw/security/advisories |
| Hetzner API docs | https://docs.hetzner.cloud/reference/cloud |

**NOOIT:** openclaw.report, willekeurige blogs, LinkedIn, Substack, AI-gegenereerde artikelen.
Als een bron niet in bovenstaande tabel staat: niet gebruiken.
