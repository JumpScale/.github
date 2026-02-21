# Research -- Actuele bronnen ophalen

Claude voert deze stap uit aan het begin van elke Install- of Maintain-sessie.
Doel: zeker weten dat de installatie gebaseerd is op de meest actuele informatie.

## Stap 1: Trigger.dev release info ophalen

Fetch: https://github.com/triggerdotdev/trigger.dev/releases

Noteer:
- Meest recente versie-nummer
- Datum van laatste release
- Breaking changes die de installatie beinvloeden
- Nieuwe environment variables of dependencies

## Stap 2: Docker image tags controleren

```bash
# Trigger.dev images staan op GitHub Container Registry
curl -s "https://api.github.com/orgs/triggerdotdev/packages?package_type=container" 2>/dev/null || echo "Controleer handmatig: https://github.com/orgs/triggerdotdev/packages"
```

Relevante images:
- `ghcr.io/triggerdotdev/trigger.dev:v3` (webapp)
- `ghcr.io/triggerdotdev/provider/docker:v3` (docker provider)
- `ghcr.io/triggerdotdev/coordinator:v3` (coordinator)

## Stap 3: Docker Compose referentie ophalen

Fetch: https://github.com/triggerdotdev/docker

Noteer:
- Huidige aanbevolen docker-compose.yml structuur
- Vereiste environment variables
- Eventuele wijzigingen ten opzichte van vorige versie

## Stap 4: Security advisories controleren

Fetch: https://github.com/triggerdotdev/trigger.dev/security/advisories

Noteer:
- Openstaande of recente CVEs
- Aanbevolen mitigaties

## Stap 5: Bevindingen samenvatten

```
Research-uitkomst:
- Trigger.dev versie: [versie] ([datum])
- Docker images: ghcr.io/triggerdotdev/trigger.dev:[tag]
- Security fixes: [samenvatting of "geen kritieke issues"]
- Docker Compose wijzigingen: [samenvatting of "geen wijzigingen"]
- Aandachtspunten: [eventuele bijzonderheden]
```

Als er kritieke security issues zijn: benoem ze expliciet en vraag of de gebruiker wil doorgaan.

## Bronnen-protocol

**Alleen deze bronnen zijn toegestaan:**

| Bron | URL |
|------|-----|
| Trigger.dev GitHub | https://github.com/triggerdotdev/trigger.dev |
| Trigger.dev Releases | https://github.com/triggerdotdev/trigger.dev/releases |
| Trigger.dev Docs | https://trigger.dev/docs |
| Trigger.dev Docker | https://github.com/triggerdotdev/docker |
| Security advisories | https://github.com/triggerdotdev/trigger.dev/security/advisories |

**NOOIT:** willekeurige blogs, LinkedIn, Substack, AI-gegenereerde artikelen.
