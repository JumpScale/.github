# Research -- Actuele bronnen ophalen

Claude voert deze stap uit aan het begin van elke Install- of Maintain-sessie.
Doel: zeker weten dat de installatie gebaseerd is op de meest actuele informatie.

## Stap 1: Trigger.dev release info ophalen

Fetch: https://github.com/triggerdotdev/trigger.dev/releases

Noteer:
- Meest recente versie-nummer (v4-beta of nieuwere stable tag)
- Datum van laatste release
- Breaking changes die de installatie beinvloeden
- Nieuwe environment variables of dependencies

## Stap 2: Docker image tags controleren

```bash
# Trigger.dev images staan op GitHub Container Registry
curl -s "https://api.github.com/orgs/triggerdotdev/packages?package_type=container" 2>/dev/null || echo "Controleer handmatig: https://github.com/orgs/triggerdotdev/packages"
```

Relevante images (v4):
- `ghcr.io/triggerdotdev/trigger.dev:v4-beta` (webapp)
- `ghcr.io/triggerdotdev/supervisor:v4-beta` (supervisor, vervangt coordinator uit v3)

Niet meer gebruikt in v4:
- ~~`ghcr.io/triggerdotdev/provider/docker:v3`~~ (vervangen door docker-proxy)
- ~~`ghcr.io/triggerdotdev/coordinator:v3`~~ (vervangen door supervisor)

## Stap 3: Docker Compose referentie ophalen

Fetch: https://github.com/triggerdotdev/trigger.dev/tree/main/hosting/docker

De v4 compose setup bestaat uit twee losse bestanden:
- `hosting/docker/webapp/docker-compose.yml` -- webapp-stack (webapp, postgres, redis, electric, clickhouse, minio)
- `hosting/docker/worker/docker-compose.yml` -- worker-stack (supervisor, docker-proxy, registry)

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
- Docker images: ghcr.io/triggerdotdev/trigger.dev:[tag], ghcr.io/triggerdotdev/supervisor:[tag]
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
| Trigger.dev Self-Host Docs | https://trigger.dev/docs/open-source-self-hosting |
| Trigger.dev Hosting (Docker) | https://github.com/triggerdotdev/trigger.dev/tree/main/hosting/docker |
| Security advisories | https://github.com/triggerdotdev/trigger.dev/security/advisories |

**NOOIT:** willekeurige blogs, LinkedIn, Substack, AI-gegenereerde artikelen.
