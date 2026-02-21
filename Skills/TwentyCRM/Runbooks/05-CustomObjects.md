# Stap 5: Custom Objects inrichten (optioneel)

## Doel
Twenty CRM uitbreiden met eigen objecten, velden en relaties via de API.

---

## 5.1 API key aanmaken [Jij moet dit doen]

```
Ga naar Twenty CRM > Settings > API > Create API Key
Geef de key een naam (bijv. "Automation") en kopieer het token.
Sla het op op de server:
```

```bash
# Claude schrijft de key direct naar bestand (NIET tonen in chat)
echo '<KEY>' > /opt/twenty-crm/.api-key
chmod 600 /opt/twenty-crm/.api-key
```

## 5.2 API key testen [Claude doet dit]

```bash
API_KEY=$(cat /opt/twenty-crm/.api-key)
curl -s -o /dev/null -w "%{http_code}" -H "Authorization: Bearer $API_KEY" \
  https://<DOMEIN>/rest/companies?limit=1
```

Verwacht: `200`

## 5.3 Custom object aanmaken [Claude doet dit]

Via de REST Metadata API:

```bash
API_KEY=$(cat /opt/twenty-crm/.api-key)
curl -s -X POST -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  "https://<DOMEIN>/rest/metadata/objects" \
  -d '{
    "nameSingular": "project",
    "namePlural": "projects",
    "labelSingular": "Project",
    "labelPlural": "Projects",
    "icon": "IconBriefcase",
    "isLabelSyncedWithName": false
  }'
```

## 5.4 Custom velden toevoegen [Claude doet dit]

Gebruik het `objectMetadataId` van het aangemaakte object:

```bash
# Voorbeeld: SELECT veld
curl -s -X POST -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  "https://<DOMEIN>/rest/metadata/fields" \
  -d '{
    "name": "status",
    "label": "Status",
    "type": "SELECT",
    "objectMetadataId": "<OBJECT_ID>",
    "options": [
      {"label": "Active", "value": "ACTIVE", "color": "green", "position": 0},
      {"label": "Completed", "value": "COMPLETED", "color": "blue", "position": 1},
      {"label": "On Hold", "value": "ON_HOLD", "color": "orange", "position": 2}
    ]
  }'
```

Beschikbare veldtypes: `TEXT`, `NUMBER`, `BOOLEAN`, `DATE_TIME`, `SELECT`, `CURRENCY`, `LINKS`, `RICH_TEXT`, `RELATION`

## 5.5 Relaties aanmaken [Claude doet dit]

Relaties worden aangemaakt via de GraphQL Metadata API (`/metadata`):

```graphql
mutation {
  createOneField(input: {
    field: {
      type: RELATION
      name: "company"
      label: "Company"
      objectMetadataId: "<SOURCE_OBJECT_ID>"
      relationCreationPayload: {
        type: MANY_TO_ONE
        targetObjectMetadataId: "<TARGET_OBJECT_ID>"
        targetFieldLabel: "Projects"
        targetFieldIcon: "IconBriefcase"
      }
    }
  }) {
    id
    name
  }
}
```

Relatie types:
- `MANY_TO_ONE` - Veel bronnen naar een doel (bijv. veel Projecten bij een Bedrijf)
- `ONE_TO_MANY` - Een bron naar veel doelen

## 5.6 Voorbeeld: JumpScale datamodel

```
Company (standaard)
├── Projects (custom, 1:N)
│   ├── status (SELECT: Active/Completed/On Hold)
│   ├── projectType (SELECT: Consultancy/Development/Managed Service)
│   ├── tier (SELECT: Essential/Professional/Premium)
│   ├── startDate (DATE_TIME)
│   ├── monthlyValue (CURRENCY)
│   ├── description (RICH_TEXT)
│   └── Opportunities (1:N relatie)
└── Directories (custom, 1:N)
    ├── url (LINKS)
    ├── directoryType (SELECT)
    ├── isActive (BOOLEAN)
    └── leadCount (NUMBER)
```

---

## Verificatie

```bash
# Alle objecten ophalen
API_KEY=$(cat /opt/twenty-crm/.api-key)
curl -s -H "Authorization: Bearer $API_KEY" \
  "https://<DOMEIN>/rest/metadata/objects" | python3 -c "
import json,sys
for o in json.load(sys.stdin)['data']['objects']:
    if o.get('isCustom'):
        print(o['labelSingular'], '-', len(o.get('fields',[])), 'fields')
"
```

Controleer ook visueel in Twenty CRM > Settings > Data Model.
