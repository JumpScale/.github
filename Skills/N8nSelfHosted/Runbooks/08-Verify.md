# 08 -- Verificatie

## Doel

Alles controleren na installatie, migratie of update.

---

## Quick check (na elke wijziging)

```bash
ssh root@<SERVER_IP> << 'EOF'
echo "=== 1. Containers ==="
docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'

echo ""
echo "=== 2. n8n Health ==="
curl -s http://localhost:5678/healthz
echo ""

echo ""
echo "=== 3. n8n Versie ==="
docker compose -f /opt/n8n/docker-compose.yml exec -T n8n n8n --version

echo ""
echo "=== 4. HTTPS ==="
curl -sI https://<N8N_DOMAIN>/ | head -3

echo ""
echo "=== 5. SSL Certificaat ==="
echo | openssl s_client -connect <N8N_DOMAIN>:443 -servername <N8N_DOMAIN> 2>/dev/null | openssl x509 -noout -dates

echo ""
echo "=== 6. Disk ==="
df -h /

echo ""
echo "=== 7. Backups ==="
ls -lh /opt/n8n/backups/ 2>/dev/null | tail -5 || echo "Geen backups"
EOF
```

---

## Volledige checklist (na installatie of migratie)

### Infrastructuur

- [ ] n8n container draait (status: Up)
- [ ] n8n-db container draait (status: Up, healthy)
- [ ] Health endpoint retourneert `{"status":"ok"}`
- [ ] HTTPS bereikbaar met geldig SSL certificaat
- [ ] HTTP redirect naar HTTPS werkt

### Security

- [ ] Poort 5678 niet bereikbaar van buitenaf
- [ ] Security headers aanwezig (X-Frame-Options, HSTS, etc.)
- [ ] Server versie niet zichtbaar in headers
- [ ] TLS 1.2+ only
- [ ] UFW actief met alleen poorten 22, 80, 443

### Functionaliteit

- [ ] Login pagina bereikbaar
- [ ] Owner account aangemaakt
- [ ] Workflows zichtbaar (als gemigreerd)
- [ ] Webhook URL bereikbaar: `curl -s https://<N8N_DOMAIN>/webhook/test`

### Backup

- [ ] Backup script aanwezig en executable
- [ ] Cron actief (dagelijks 03:00 UTC)
- [ ] Eerste backup geslaagd

### Migratie (alleen als van toepassing)

- [ ] Alle workflows geimporteerd
- [ ] Workflows staan op inactive
- [ ] Credentials opnieuw ingesteld
- [ ] Webhook URLs bijgewerkt in externe services

---

## Afsluitende samenvatting

```
n8n installatie geverifieerd.

Gegevens:
- URL: https://[domein]
- Webhook base: https://[domein]/webhook/
- Versie: [versie]
- Backup: dagelijks 03:00 UTC, 7 dagen retentie

Bewaar veilig:
- Server IP
- n8n login-gegevens
- Encryption key (in /opt/n8n/.env)
```
