# 04 -- Security hardening

## Doel

Trigger.dev installatie beveiligen tegen ongeautoriseerde toegang.

---

## Stap 1: Webapp alleen via localhost [Claude doet dit]

Trigger.dev webapp mag niet direct bereikbaar zijn op poort 3040. Alleen via de reverse proxy.

```bash
# Controleer docker-compose.yml
ssh root@<SERVER_IP> "grep -A2 ports /opt/trigger/docker-compose.yml"
```

Verwacht: `127.0.0.1:3040:3030` (niet `0.0.0.0:3040:3030` of `3040:3030`).

## Stap 2: Firewall controleren [Claude doet dit]

```bash
ssh root@<SERVER_IP> "ufw status verbose"
```

Alleen deze poorten open:
- 22/tcp (SSH)
- 80/tcp (HTTP, redirect naar HTTPS)
- 443/tcp (HTTPS)

Poort 3040 mag NIET open staan.

## Stap 3: Security headers [Claude doet dit]

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

## Stap 4: Server tokens verbergen [Claude doet dit]

```bash
# Voeg toe aan nginx.conf http block:
ssh root@<SERVER_IP> "sed -i '/http {/a\\    server_tokens off;' /etc/nginx/nginx.conf"
ssh root@<SERVER_IP> "nginx -t && systemctl reload nginx"
```

## Stap 5: TLS configuratie [Claude doet dit]

Alleen TLS 1.2 en 1.3 toestaan:

```bash
ssh root@<SERVER_IP> "grep ssl_protocols /etc/nginx/nginx.conf"
```

Verwacht: `ssl_protocols TLSv1.2 TLSv1.3;`

## Stap 6: Database niet blootstellen [Claude doet dit]

```bash
# PostgreSQL mag alleen intern bereikbaar zijn
ssh root@<SERVER_IP> "ss -tlnp | grep 5432"
```

Verwacht: geen output (database alleen via Docker netwerk bereikbaar).

## Stap 7: Redis niet blootstellen [Claude doet dit]

```bash
# Redis mag alleen intern bereikbaar zijn
ssh root@<SERVER_IP> "ss -tlnp | grep 6379"
```

Verwacht: geen output (Redis alleen via Docker netwerk bereikbaar).

## Stap 8: Docker socket beveiligen [Claude doet dit]

De docker-provider service heeft toegang tot de Docker socket. Controleer dat alleen de juiste container deze mount heeft:

```bash
ssh root@<SERVER_IP> "docker inspect trigger-docker-provider --format '{{range .Mounts}}{{.Source}} -> {{.Destination}}{{println}}{{end}}'"
```

Verwacht: alleen `/var/run/docker.sock -> /var/run/docker.sock`.

Zorg dat de Docker socket alleen leesbaar is voor root:

```bash
ssh root@<SERVER_IP> "ls -la /var/run/docker.sock"
```

Verwacht: `srw-rw----` met eigenaar `root:docker`.

## Verificatie

```bash
# Headers controleren
curl -sI https://<TRIGGER_DOMAIN>/ | grep -iE 'x-frame|x-content|referrer|strict-transport|server'

# Poort 3040 niet bereikbaar van buitenaf
curl -s --connect-timeout 3 http://<TRIGGER_DOMAIN>:3040/ && echo "ONVEILIG" || echo "OK - niet bereikbaar"

# Database poort niet bereikbaar van buitenaf
curl -s --connect-timeout 3 http://<TRIGGER_DOMAIN>:5432/ && echo "ONVEILIG" || echo "OK - niet bereikbaar"
```

Verwacht:
- Security headers aanwezig
- Poort 3040 niet bereikbaar van buitenaf
- Poort 5432 niet bereikbaar van buitenaf
- Poort 6379 niet bereikbaar van buitenaf
- Geen server versie zichtbaar
- Docker socket correct beveiligd
