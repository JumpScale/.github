# 04 -- Workflows migreren

## Doel

Workflows van n8n cloud exporteren en importeren in self-hosted instance.

**Let op:** Credentials worden NIET mee gemigreerd. Die moet je opnieuw instellen (zie `05-Credentials.md`).

---

## Stap 1: Workflows exporteren uit cloud [Claude doet dit]

Benodigdheden:
- n8n cloud URL (bijv. `https://mijnbedrijf.app.n8n.cloud`)
- n8n cloud API key (Settings > API)

```python
import requests, json, os

CLOUD_URL = "<CLOUD_URL>"
API_KEY = "<API_KEY>"
OUTPUT_DIR = "/opt/n8n/import"

os.makedirs(OUTPUT_DIR, exist_ok=True)
headers = {"X-N8N-API-KEY": API_KEY}

r = requests.get(f"{CLOUD_URL}/api/v1/workflows?limit=250", headers=headers)
workflows = [w for w in r.json()["data"] if not w.get("tags", [])]  # skip archived

for wf in workflows:
    detail = requests.get(f"{CLOUD_URL}/api/v1/workflows/{wf['id']}", headers=headers).json()
    with open(f"{OUTPUT_DIR}/{wf['id']}.json", "w") as f:
        json.dump(detail, f, indent=2)

print(f"Geexporteerd: {len(workflows)} workflows")
```

## Stap 2: Opschonen [Claude doet dit]

Cloud exports bevatten referenties die FOREIGN KEY fouten veroorzaken bij import. Alleen essentials behouden:

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
    for node in clean.get("nodes", []):
        if "credentials" in node:
            for cred_val in node["credentials"].values():
                if isinstance(cred_val, dict):
                    cred_val.pop("id", None)
    with open(f"{OUTPUT_DIR}/{os.path.basename(f)}", "w") as fh:
        json.dump(clean, fh, indent=2)

print(f"Opgeschoond: {len(glob.glob(f'{OUTPUT_DIR}/*.json'))} workflows")
```

## Stap 3: Importeren via CLI [Claude doet dit]

```bash
# Kopieer opgeschoonde workflows naar n8n container
ssh root@<SERVER_IP> << 'EOF'
docker cp /opt/n8n/import_minimal n8n:/home/node/import_minimal
docker compose -f /opt/n8n/docker-compose.yml exec -T n8n n8n import:workflow --separate --input=/home/node/import_minimal/
EOF
```

## Stap 4: Verificatie [Claude doet dit]

```bash
ssh root@<SERVER_IP> "docker compose -f /opt/n8n/docker-compose.yml exec -T n8n n8n list:workflow"
```

## Resultaat

```
Migratie voltooid:
- [aantal] workflows geimporteerd
- Alle workflows staan op INACTIVE
- Credentials moeten opnieuw ingesteld worden

Volgende stap: 05-Credentials.md
```
