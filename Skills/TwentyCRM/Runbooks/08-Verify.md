# Stap 8: Verificatie-checklist

## Doel
Alles controleren voordat de installatie als voltooid wordt beschouwd.

---

## Checklist

Voer elke check uit. Markeer als geslaagd of gefaald.

### Basis (altijd)

```bash
# 1. Twenty CRM containers draaien
docker compose -f /opt/twenty-crm/docker-compose.yml ps
# Verwacht: server (healthy), worker, db (healthy), redis (healthy)

# 2. Twenty CRM bereikbaar via HTTPS
curl -sI https://<DOMEIN> | head -3
# Verwacht: HTTP/1.1 200 OK

# 3. SSL certificaat geldig
echo | openssl s_client -connect <DOMEIN>:443 -servername <DOMEIN> 2>/dev/null | openssl x509 -noout -dates
# Verwacht: notAfter in de toekomst

# 4. HTTP redirect naar HTTPS
curl -sI http://<DOMEIN> | head -3
# Verwacht: 301 redirect naar https://

# 5. Security headers aanwezig
curl -sI https://<DOMEIN> | grep -ciE "x-frame|strict-transport|x-content|referrer|permissions"
# Verwacht: 5 of meer

# 6. Server versie verborgen
curl -sI https://<DOMEIN> | grep -i server
# Verwacht: "Server: nginx" (GEEN versienummer)

# 7. Firewall actief
ufw status | head -3
# Verwacht: Status: active

# 8. Fail2ban actief
systemctl is-active fail2ban
# Verwacht: active

# 9. Backup procedure werkt
ls -lh /opt/backups/ | tail -3
# Verwacht: recente bestanden met redelijke grootte

# 10. Cron backup ingesteld
crontab -l | grep backup
# Verwacht: dagelijkse backup regel
```

### n8n (indien geinstalleerd)

```bash
# 11. n8n container draait
docker compose -f /opt/n8n/docker-compose.yml ps
# Verwacht: n8n (Up)

# 12. n8n bereikbaar via HTTPS
curl -sI https://n8n.<DOMEIN> | head -3
# Verwacht: HTTP/1.1 200 OK

# 13. n8n SSL certificaat geldig
echo | openssl s_client -connect n8n.<DOMEIN>:443 -servername n8n.<DOMEIN> 2>/dev/null | openssl x509 -noout -dates
```

### Custom Objects (indien aangemaakt)

```bash
# 14. Custom objects zichtbaar via API
API_KEY=$(cat /opt/twenty-crm/.api-key)
curl -s -H "Authorization: Bearer $API_KEY" "https://<DOMEIN>/rest/metadata/objects" | \
  python3 -c "import json,sys; [print('  ' + o['labelSingular']) for o in json.load(sys.stdin)['data']['objects'] if o.get('isCustom')]"
# Verwacht: aangemaakte objecten worden getoond

# 15. API authenticatie werkt
curl -s -o /dev/null -w "%{http_code}" -H "Authorization: Bearer $API_KEY" "https://<DOMEIN>/rest/companies?limit=1"
# Verwacht: 200

# 16. API ZONDER auth wordt geweigerd
curl -s -o /dev/null -w "%{http_code}" "https://<DOMEIN>/rest/companies?limit=1"
# Verwacht: 403
```

---

## Samenvatting aan gebruiker

Presenteer de resultaten als een overzicht:

```
Verificatie voltooid:

Basis:
[✓/✗] Twenty CRM containers draaien
[✓/✗] HTTPS bereikbaar
[✓/✗] SSL certificaat geldig
[✓/✗] HTTP redirect naar HTTPS
[✓/✗] Security headers (X/5)
[✓/✗] Server versie verborgen
[✓/✗] Firewall actief
[✓/✗] Fail2ban actief
[✓/✗] Backup procedure
[✓/✗] Cron backup

n8n:
[✓/✗] Container draait
[✓/✗] HTTPS bereikbaar
[✓/✗] SSL geldig

Custom Objects:
[✓/✗] Objecten zichtbaar
[✓/✗] API authenticatie
[✓/✗] Ongeautoriseerde toegang geblokkeerd
```

Bij gefaalde checks: leg uit wat er mis is en bied aan om het te fixen.
