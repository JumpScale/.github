# 06 -- Security hardening

## Doel

n8n installatie beveiligen tegen ongeautoriseerde toegang.

---

## Stap 1: n8n alleen via localhost [Claude doet dit]

n8n mag niet direct bereikbaar zijn op poort 5678. Alleen via de reverse proxy.

```bash
# Controleer docker-compose.yml
ssh root@<SERVER_IP> "grep -A2 ports /opt/n8n/docker-compose.yml"
```

Verwacht: `127.0.0.1:5678:5678` (niet `0.0.0.0:5678:5678` of `5678:5678`).

## Stap 2: Firewall controleren [Claude doet dit]

```bash
ssh root@<SERVER_IP> "ufw status verbose"
```

Alleen deze poorten open:
- 22/tcp (SSH)
- 80/tcp (HTTP, redirect naar HTTPS)
- 443/tcp (HTTPS)

Poort 5678 mag NIET open staan.

## Stap 3: Security headers [Claude doet dit]

### nginx

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

### Caddy

Caddy stuurt de meeste security headers automatisch mee, inclusief HSTS.

## Stap 4: Server tokens verbergen [Claude doet dit]

### nginx

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

## Verificatie

```bash
# Headers controleren
curl -sI https://<N8N_DOMAIN>/ | grep -iE 'x-frame|x-content|referrer|strict-transport|server'

# Poort 5678 niet bereikbaar van buitenaf
curl -s --connect-timeout 3 http://<N8N_DOMAIN>:5678/ && echo "ONVEILIG" || echo "OK - niet bereikbaar"
```

Verwacht:
- Security headers aanwezig
- Poort 5678 niet bereikbaar van buitenaf
- Geen server versie zichtbaar
