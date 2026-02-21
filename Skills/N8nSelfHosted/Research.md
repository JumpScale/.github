# Research -- Actuele bronnen ophalen

Claude voert deze stap uit aan het begin van elke Install-, Migrate- of Maintain-sessie.
Doel: zeker weten dat de installatie gebaseerd is op de meest actuele informatie.

## Stap 1: n8n release info ophalen

Fetch: https://github.com/n8n-io/n8n/releases

Noteer:
- Meest recente versie-nummer
- Datum van laatste release
- Breaking changes die de installatie beinvloeden
- Nieuwe environment variables of dependencies

## Stap 2: Docker image tags controleren

```bash
curl -s https://hub.docker.com/v2/repositories/n8nio/n8n/tags/?page_size=5 | python3 -c "import json,sys; [print(t['name'], t['last_updated'][:10]) for t in json.load(sys.stdin)['results']]"
```

Noteer: beschikbare tags en welke het meest recent is.

## Stap 3: Security advisories controleren

Fetch: https://github.com/n8n-io/n8n/security/advisories

Noteer:
- Openstaande of recente CVEs
- Aanbevolen mitigaties

## Stap 4: Bevindingen samenvatten

```
Research-uitkomst:
- n8n versie: [versie] ([datum])
- Docker image: n8nio/n8n:[tag]
- Security fixes: [samenvatting of "geen kritieke issues"]
- Aandachtspunten: [eventuele bijzonderheden]
```

Als er kritieke security issues zijn: benoem ze expliciet en vraag of de gebruiker wil doorgaan.

## Bronnen-protocol

**Alleen deze bronnen zijn toegestaan:**

| Bron | URL |
|------|-----|
| n8n GitHub | https://github.com/n8n-io/n8n |
| n8n Releases | https://github.com/n8n-io/n8n/releases |
| n8n Docs | https://docs.n8n.io |
| Security advisories | https://github.com/n8n-io/n8n/security/advisories |
| Docker Hub | https://hub.docker.com/r/n8nio/n8n |

**NOOIT:** willekeurige blogs, LinkedIn, Substack, AI-gegenereerde artikelen.
