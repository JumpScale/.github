# 04 -- Security hardening

## Doel

Trigger.dev v4 installatie beveiligen tegen ongeautoriseerde toegang.

---

## Stap 1: Webapp alleen via localhost [Claude doet dit]

Trigger.dev webapp mag niet direct bereikbaar zijn op poort 8030. Alleen via de reverse proxy.

```bash
# Controleer docker-compose.yml
ssh root@<SERVER_IP> "grep -A2 ports /opt/trigger-dev/webapp/docker-compose.yml | head -6"
```

Verwacht: `127.0.0.1:8030:3000` (niet `0.0.0.0:8030:3000` of `8030:3000`).

## Stap 2: Alle services op localhost [Claude doet dit]

In v4 draaien meer services die niet extern bereikbaar mogen zijn. Controleer dat alle PUBLISH_IP variabelen op 127.0.0.1 staan:

```bash
ssh root@<SERVER_IP> "grep PUBLISH_IP /opt/trigger-dev/.env"
```

Verwacht: alle IP's zijn `127.0.0.1`.

Poorten die NIET extern bereikbaar mogen zijn:
- 8030 (webapp)
- 5433 (postgres)
- 6389 (redis)
- 8123 (clickhouse)
- 9000/9001 (minio)
- 3060 (electric)

## Stap 3: Firewall controleren [Claude doet dit]

```bash
ssh root@<SERVER_IP> "ufw status verbose"
```

Alleen deze poorten open:
- 22/tcp (SSH)
- 80/tcp (HTTP, redirect naar HTTPS)
- 443/tcp (HTTPS)

Geen enkele Trigger.dev poort mag open staan in UFW.

## Stap 4: Security headers [Claude doet dit]

```bash
ssh root@<SERVER_IP> "cat > /etc/nginx/conf.d/security-headers.conf" << 'EOF'
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-Content-Type-Options "nosniff" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
add_header Permissions-Policy "camera=(), microphone=(), geolocation=()" always;
EOF

ssh root@<SERVER_IP> "nginx -t && systemctl reload nginx"
```

## Stap 5: Server tokens verbergen [Claude doet dit]

```bash
# Voeg toe aan nginx.conf http block:
ssh root@<SERVER_IP> "sed -i '/http {/a\\    server_tokens off;' /etc/nginx/nginx.conf"
ssh root@<SERVER_IP> "nginx -t && systemctl reload nginx"
```

## Stap 6: TLS configuratie [Claude doet dit]

Alleen TLS 1.2 en 1.3 toestaan:

```bash
ssh root@<SERVER_IP> "grep ssl_protocols /etc/nginx/nginx.conf"
```

Verwacht: `ssl_protocols TLSv1.2 TLSv1.3;`

## Stap 7: Database niet blootstellen [Claude doet dit]

```bash
# PostgreSQL mag alleen intern bereikbaar zijn
ssh root@<SERVER_IP> "ss -tlnp | grep 5432"
```

Verwacht: alleen binding op 127.0.0.1 (niet 0.0.0.0).

## Stap 8: Redis niet blootstellen [Claude doet dit]

```bash
# Redis mag alleen intern bereikbaar zijn
ssh root@<SERVER_IP> "ss -tlnp | grep 6379"
```

Verwacht: alleen binding op 127.0.0.1 (niet 0.0.0.0).

## Stap 9: ClickHouse niet blootstellen [Claude doet dit]

```bash
# ClickHouse mag alleen intern bereikbaar zijn
ssh root@<SERVER_IP> "ss -tlnp | grep 8123"
```

Verwacht: alleen binding op 127.0.0.1 (niet 0.0.0.0).

## Stap 10: MinIO niet blootstellen [Claude doet dit]

```bash
# MinIO mag alleen intern bereikbaar zijn
ssh root@<SERVER_IP> "ss -tlnp | grep -E '9000|9001'"
```

Verwacht: alleen binding op 127.0.0.1 (niet 0.0.0.0).

## Stap 11: Docker socket beveiligen [Claude doet dit]

De docker-proxy service heeft toegang tot de Docker socket. Controleer dat alleen de juiste container deze mount heeft:

```bash
ssh root@<SERVER_IP> "docker inspect trigger-docker-proxy --format '{{range .Mounts}}{{.Source}} -> {{.Destination}}{{println}}{{end}}'"
```

Verwacht: alleen `/var/run/docker.sock -> /var/run/docker.sock`.

Zorg dat de Docker socket alleen leesbaar is voor root:

```bash
ssh root@<SERVER_IP> "ls -la /var/run/docker.sock"
```

Verwacht: `srw-rw----` met eigenaar `root:docker`.

## Stap 12: GitHub Container Registry credentials [Claude doet dit]

Controleer dat ghcr.io credentials niet in plaintext logs terechtkomen:

```bash
ssh root@<SERVER_IP> "cat /root/.docker/config.json | python3 -c 'import json,sys; d=json.load(sys.stdin); print(\"OK - credsStore\" if \"credsStore\" in d else \"WAARSCHUWING - plaintext credentials in config.json\")' 2>/dev/null || echo 'Docker config niet gevonden'"
```

## Verificatie

```bash
# Headers controleren
curl -sI https://<TRIGGER_DOMAIN>/ | grep -iE 'x-frame|x-content|referrer|strict-transport|server'

# Poort 8030 niet bereikbaar van buitenaf
curl -s --connect-timeout 3 http://<TRIGGER_DOMAIN>:8030/ && echo "ONVEILIG" || echo "OK - niet bereikbaar"

# Database poort niet bereikbaar van buitenaf
curl -s --connect-timeout 3 http://<TRIGGER_DOMAIN>:5433/ && echo "ONVEILIG" || echo "OK - niet bereikbaar"

# ClickHouse poort niet bereikbaar van buitenaf
curl -s --connect-timeout 3 http://<TRIGGER_DOMAIN>:8123/ && echo "ONVEILIG" || echo "OK - niet bereikbaar"

# MinIO poort niet bereikbaar van buitenaf
curl -s --connect-timeout 3 http://<TRIGGER_DOMAIN>:9000/ && echo "ONVEILIG" || echo "OK - niet bereikbaar"
```

Verwacht:
- Security headers aanwezig
- Poort 8030 niet bereikbaar van buitenaf
- Poort 5433 niet bereikbaar van buitenaf
- Poort 6389 niet bereikbaar van buitenaf
- Poort 8123 niet bereikbaar van buitenaf
- Poort 9000/9001 niet bereikbaar van buitenaf
- Geen server versie zichtbaar
- Docker socket correct beveiligd
