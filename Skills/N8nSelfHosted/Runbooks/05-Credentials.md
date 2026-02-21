# 05 -- Credentials instellen

## Doel

Credentials opnieuw instellen in self-hosted n8n na migratie. n8n versleutelt credentials per instance, dus ze kunnen niet worden overgezet.

---

## Stap 1: Overzicht maken [Claude doet dit]

Claude analyseert alle geimporteerde workflows en maakt een lijst van benodigde credentials:

```bash
# Alle unieke credential types uit workflows halen
ssh root@<SERVER_IP> << 'EOF'
docker compose -f /opt/n8n/docker-compose.yml exec -T n8n \
  n8n export:workflow --all --output=/tmp/all-workflows.json 2>/dev/null

# Parse credential types
python3 -c "
import json, glob
creds = set()
for f in glob.glob('/opt/n8n/import_minimal/*.json'):
    with open(f) as fh:
        wf = json.load(fh)
    for node in wf.get('nodes', []):
        if 'credentials' in node:
            for name, val in node['credentials'].items():
                creds.add(name)
for c in sorted(creds):
    print(f'  - {c}')
"
EOF
```

## Stap 2: Per credential type instellen [Jij moet dit doen]

Ga naar: `https://<N8N_DOMAIN>/credentials`

Klik op **Add Credential** en stel in:

### Veelvoorkomende credential types

| Type | Wat je nodig hebt | Waar te vinden |
|------|-------------------|----------------|
| **HTTP Header Auth** | Header naam + waarde | API documentatie van de service |
| **OAuth2** | Client ID + Client Secret | Developer portal van de service |
| **API Key** | API key | Settings/API pagina van de service |
| **Basic Auth** | Gebruikersnaam + wachtwoord | Account instellingen |

### Tips

- **Naam exact hetzelfde** als in de cloud instance. Workflows refereren naar credentials op naam.
- **OAuth2:** Je moet opnieuw autoriseren. De callback URL verandert naar je self-hosted domein.
- **Test elke credential** via de "Test" knop in n8n voordat je workflows activeert.

## Stap 3: Workflows activeren [Jij moet dit doen]

Na het instellen van credentials, activeer workflows een voor een:

1. Open workflow in de editor
2. Controleer of alle nodes groene credentials hebben (geen rode waarschuwingen)
3. Test met de "Execute Workflow" knop
4. Zet op Active als alles werkt

## Verificatie

```
Credentials check:
- [ ] Alle benodigde credentials aangemaakt
- [ ] Elke credential getest via n8n Test-knop
- [ ] Workflows geactiveerd die je nodig hebt
- [ ] Webhook URLs bijgewerkt in externe services
```
