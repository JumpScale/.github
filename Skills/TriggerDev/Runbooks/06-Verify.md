# 06 -- Verificatie

## Doel

Alles controleren na installatie of update.

---

## Quick check (na elke wijziging)

```bash
ssh root@<SERVER_IP> << 'EOF'
echo "=== 1. Containers ==="
docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}' | grep -E 'trigger|electric'

echo ""
echo "=== 2. Webapp bereikbaar ==="
curl -s -o /dev/null -w "HTTP %{http_code}" http://localhost:3040/
echo ""

echo ""
echo "=== 3. HTTPS ==="
curl -sI https://<TRIGGER_DOMAIN>/ | head -3

echo ""
echo "=== 4. SSL Certificaat ==="
echo | openssl s_client -connect <TRIGGER_DOMAIN>:443 -servername <TRIGGER_DOMAIN> 2>/dev/null | openssl x509 -noout -dates

echo ""
echo "=== 5. Disk ==="
df -h /

echo ""
echo "=== 6. RAM ==="
free -h

echo ""
echo "=== 7. Backups ==="
ls -lh /opt/trigger/backups/ 2>/dev/null | tail -5 || echo "Geen backups"
EOF
```

---

## Volledige checklist (na installatie)

### Infrastructuur

- [ ] webapp container draait (status: Up)
- [ ] postgres container draait (status: Up, healthy)
- [ ] redis container draait (status: Up)
- [ ] coordinator container draait (status: Up)
- [ ] docker-provider container draait (status: Up)
- [ ] electric container draait (status: Up)
- [ ] Webapp bereikbaar op localhost:3040
- [ ] HTTPS bereikbaar met geldig SSL certificaat
- [ ] HTTP redirect naar HTTPS werkt

### Security

- [ ] Poort 3040 niet bereikbaar van buitenaf
- [ ] Poort 5432 niet bereikbaar van buitenaf
- [ ] Poort 6379 niet bereikbaar van buitenaf
- [ ] Security headers aanwezig (X-Frame-Options, HSTS, etc.)
- [ ] Server versie niet zichtbaar in headers
- [ ] TLS 1.2+ only
- [ ] UFW actief met alleen poorten 22, 80, 443
- [ ] Docker socket alleen gemount in docker-provider

### Functionaliteit

- [ ] Dashboard bereikbaar via browser
- [ ] Magic link login werkt (link verschijnt in container logs)
- [ ] Account aangemaakt

### Backup

- [ ] Backup script aanwezig en executable
- [ ] Cron actief (dagelijks 03:00 UTC)
- [ ] Eerste backup geslaagd

---

## Afsluitende samenvatting

```
Trigger.dev installatie geverifieerd.

Gegevens:
- URL: https://[domein]
- API URL (voor SDK): https://[domein]
- Backup: dagelijks 03:00 UTC, 7 dagen retentie

Bewaar veilig:
- Server IP
- Login e-mailadres
- Secrets (in /opt/trigger/.env)

SDK configuratie:
- TRIGGER_API_URL=https://[domein]
- Deploy tasks: npx trigger.dev@latest deploy
```
