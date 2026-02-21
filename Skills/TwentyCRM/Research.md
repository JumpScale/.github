# Research -- Actuele bronnen ophalen

Claude voert deze stap uit aan het begin van elke Install- of Maintain-sessie.
Doel: zeker weten dat de installatie gebaseerd is op de meest actuele informatie.

## Stap 1: Twenty CRM release info ophalen

Fetch: https://github.com/twentyhq/twenty/releases

Noteer:
- Meest recente versie-nummer
- Datum van laatste release
- Breaking changes die de installatie beinvloeden
- Nieuwe environment variables of dependencies

## Stap 2: Docker image tags controleren

```bash
curl -s https://hub.docker.com/v2/repositories/twentycrm/twenty/tags/?page_size=5 | python3 -c "import json,sys; [print(t['name'], t['last_updated'][:10]) for t in json.load(sys.stdin)['results']]"
```

Noteer: beschikbare tags en welke het meest recent is.

## Stap 3: Security advisories controleren

Fetch: https://github.com/twentyhq/twenty/security/advisories

Noteer:
- Openstaande of recente CVEs
- Aanbevolen mitigaties
- Of de huidige installatie (indien Maintain) kwetsbaar is

## Stap 4: n8n versie controleren (indien van toepassing)

Fetch: https://github.com/n8n-io/n8n/releases

Noteer:
- Meest recente versie
- Security fixes

## Stap 5: Bevindingen samenvatten

Na het ophalen van bovenstaande informatie, geef de gebruiker een korte samenvatting:

```
Research-uitkomst:
- Twenty CRM versie: [versie] ([datum])
- Docker image: twentycrm/twenty:[tag]
- Security fixes: [samenvatting of "geen kritieke issues"]
- n8n versie: [versie] (indien van toepassing)
- Aandachtspunten: [eventuele bijzonderheden]
```

Als er kritieke security issues zijn: benoem ze expliciet en vraag of de gebruiker wil doorgaan.

## Bronnen-protocol

**Alleen deze bronnen zijn toegestaan:**

| Bron | URL |
|------|-----|
| Twenty CRM GitHub | https://github.com/twentyhq/twenty |
| Twenty Releases | https://github.com/twentyhq/twenty/releases |
| Twenty Docs | https://docs.twenty.com |
| Security advisories | https://github.com/twentyhq/twenty/security/advisories |
| Docker Hub | https://hub.docker.com/r/twentycrm/twenty |
| n8n GitHub | https://github.com/n8n-io/n8n |
| n8n Releases | https://github.com/n8n-io/n8n/releases |

**NOOIT:** willekeurige blogs, LinkedIn, Substack, AI-gegenereerde artikelen.
Als een bron niet in bovenstaande tabel staat: niet gebruiken.
