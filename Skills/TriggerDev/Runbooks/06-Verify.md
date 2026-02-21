# 06 -- Verificatie

## Doel

Alles controleren na installatie of update.

---

## Quick check (na elke wijziging)

```bash
ssh root@<SERVER_IP> << 'EOF'
echo "=== 1. Containers (verwacht: 9 stuks) ==="
docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}' | grep -E 'trigger|electric'

echo ""
echo "=== 2. Webapp bereikbaar ==="
curl -s -o /dev/null -w "HTTP %{http_code}" http://localhost:8030/
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
ls -lh /opt/trigger-dev/backups/ 2>/dev/null | tail -5 || echo "Geen backups"
EOF
```

---

## Volledige checklist (na installatie)

### Infrastructuur (9 containers)

**Webapp stack:**
- [ ] webapp container draait (status: Up)
- [ ] postgres container draait (status: Up, healthy)
- [ ] redis container draait (status: Up)
- [ ] electric container draait (status: Up)
- [ ] clickhouse container draait (status: Up)
- [ ] minio container draait (status: Up)

**Worker stack:**
- [ ] supervisor container draait (status: Up)
- [ ] docker-proxy container draait (status: Up)
- [ ] registry container draait (status: Up) -- of ghcr.io geconfigureerd

**Connectiviteit:**
- [ ] Webapp bereikbaar op localhost:8030
- [ ] HTTPS bereikbaar met geldig SSL certificaat
- [ ] HTTP redirect naar HTTPS werkt

### Security

- [ ] Poort 8030 niet bereikbaar van buitenaf
- [ ] Poort 5433 niet bereikbaar van buitenaf
- [ ] Poort 6389 niet bereikbaar van buitenaf
- [ ] Poort 8123 niet bereikbaar van buitenaf (ClickHouse)
- [ ] Poort 9000/9001 niet bereikbaar van buitenaf (MinIO)
- [ ] Security headers aanwezig (X-Frame-Options, HSTS, etc.)
- [ ] Server versie niet zichtbaar in headers
- [ ] TLS 1.2+ only
- [ ] UFW actief met alleen poorten 22, 80, 443
- [ ] Docker socket alleen gemount in docker-proxy
- [ ] Alle PUBLISH_IP variabelen op 127.0.0.1

### Functionaliteit

- [ ] Dashboard bereikbaar via browser
- [ ] Magic link login werkt (link verschijnt in container logs)
- [ ] Account aangemaakt
- [ ] ghcr.io login werkt op VPS (`docker login ghcr.io`)

### Backup

- [ ] Backup script aanwezig en executable
- [ ] Cron actief (dagelijks 03:00 UTC) of via Twenty CRM backup
- [ ] Eerste backup geslaagd
- [ ] Container naam: trigger-postgres-1 (user: postgres)

---

## Afsluitende samenvatting

```
Trigger.dev v4 installatie geverifieerd.

Gegevens:
- URL: https://[domein]
- API URL (voor SDK): https://[domein]
- Backup: dagelijks 03:00 UTC, 7 dagen retentie
- Containers: 9 stuks (webapp, supervisor, docker-proxy, postgres, redis, electric, clickhouse, minio, registry)
- Management: /opt/trigger-dev/trigger-ctl.sh {up|down|restart|logs|status}

Bewaar veilig:
- Server IP
- Login e-mailadres
- Secrets (in /opt/trigger-dev/.env)
- GitHub PAT voor ghcr.io

SDK configuratie:
- TRIGGER_API_URL=https://[domein]
- Deploy tasks (vanaf VPS): npx trigger.dev@latest deploy --api-url https://[domein]
```
