# Migrate Workflow

Migratie van n8n cloud naar self-hosted. Workflows worden geexporteerd, opgeschoond en geimporteerd.

```
We gaan je n8n cloud workflows overzetten naar je self-hosted instance.
Credentials worden NIET mee gemigreerd -- die moet je opnieuw instellen.
```

---

## Fase 0: Vereisten checken

### Check 1 -- Self-hosted n8n draait [Claude doet dit]

```bash
ssh root@<SERVER_IP> "docker ps --format '{{.Names}}' | grep n8n"
```

- **Draait:** Ga door.
- **Draait niet:** Voer eerst de Install workflow uit.

### Check 2 -- Cloud API key [Jij moet dit doen]

```
Heb je een API key voor je n8n cloud instance?

Ga naar: https://<jouw-instance>.app.n8n.cloud/settings/api
Maak een API key aan met volledige rechten.
```

### Check 3 -- Cloud instance URL [Jij moet dit doen]

```
Wat is de URL van je n8n cloud instance?
Bijvoorbeeld: https://mijnbedrijf.app.n8n.cloud
```

---

## Fase 1: Research

Lees en volg: `../Research.md`

---

## Fase 2: Export uit cloud

### Stap 1 -- Workflows ophalen [Claude doet dit]

```python
import requests, json, os

CLOUD_URL = "<CLOUD_URL>"
API_KEY = "<API_KEY>"
OUTPUT_DIR = "/opt/n8n/import"

os.makedirs(OUTPUT_DIR, exist_ok=True)
headers = {"X-N8N-API-KEY": API_KEY}

r = requests.get(f"{CLOUD_URL}/api/v1/workflows?limit=250", headers=headers)
workflows = r.json()["data"]

exported = 0
for wf in workflows:
    detail = requests.get(f"{CLOUD_URL}/api/v1/workflows/{wf['id']}", headers=headers).json()
    with open(f"{OUTPUT_DIR}/{wf['id']}.json", "w") as f:
        json.dump(detail, f, indent=2)
    exported += 1

print(f"Geexporteerd: {exported} workflows")
```

### Stap 2 -- Opschonen [Claude doet dit]

Cloud exports bevatten referenties naar cloud-specifieke objecten (shared users, tags, projects). Die veroorzaken FOREIGN KEY fouten bij import. Alleen essentials behouden:

```python
import json, glob, os

KEEP = {"name", "nodes", "connections", "settings", "staticData", "pinData"}
INPUT_DIR = "/opt/n8n/import"
OUTPUT_DIR = "/opt/n8n/import_minimal"

os.makedirs(OUTPUT_DIR, exist_ok=True)

for f in glob.glob(f"{INPUT_DIR}/*.json"):
    with open(f) as fh:
        wf = json.load(fh)
    clean = {k: v for k, v in wf.items() if k in KEEP}
    clean["active"] = False
    # Strip credential IDs (niet geldig op nieuwe instance)
    for node in clean.get("nodes", []):
        if "credentials" in node:
            for cred_val in node["credentials"].values():
                if isinstance(cred_val, dict):
                    cred_val.pop("id", None)
    with open(f"{OUTPUT_DIR}/{os.path.basename(f)}", "w") as fh:
        json.dump(clean, fh, indent=2)

print(f"Opgeschoond: {len(glob.glob(f'{OUTPUT_DIR}/*.json'))} workflows")
```

---

## Fase 3: Import in self-hosted

### Stap 1 -- Importeren via CLI [Claude doet dit]

```bash
ssh root@<SERVER_IP> << 'EOF'
cd /opt/n8n
docker compose exec -T n8n n8n import:workflow --separate --input=/home/node/import_minimal/
EOF
```

### Stap 2 -- Verificatie [Claude doet dit]

```bash
ssh root@<SERVER_IP> "docker compose -f /opt/n8n/docker-compose.yml exec -T n8n n8n list:workflow" | head -30
```

### Samenvatting

```
Migratie voltooid:
- [aantal] workflows geexporteerd uit cloud
- [aantal] workflows geimporteerd in self-hosted
- Alle workflows staan op INACTIVE (handmatig activeren na credentials)

Volgende stap: credentials instellen per workflow.
```

---

## Fase 4: Credentials instellen

Lees en volg: `../Runbooks/05-Credentials.md`

---

## Fase 5: Verificatie

Lees en volg: `../Runbooks/08-Verify.md`

---

## Afsluiting

```
Migratie voltooid.

Samenvatting:
- [aantal] workflows overgezet
- Credentials opnieuw ingesteld
- Webhook base URL: https://[domein]/webhook/

Volgende stappen:
- Webhook URLs bijwerken in externe services (formulieren, apps)
- Workflows activeren die je nodig hebt
- Cloud instance uitschakelen (pas als alles werkt)
```
