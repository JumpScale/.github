# 03 -- Reverse Proxy + SSL

## Doel

n8n bereikbaar via HTTPS met automatisch SSL-certificaat.

---

## Optie 1: nginx + certbot

### Stap 1: nginx installeren

```bash
ssh root@<SERVER_IP> << 'EOF'
apt install -y nginx certbot python3-certbot-nginx
systemctl enable nginx
EOF
```

### Stap 2: nginx configuratie

```bash
ssh root@<SERVER_IP> "cat > /etc/nginx/sites-available/n8n" << 'EOF'
server {
    listen 80;
    server_name <N8N_DOMAIN>;

    location / {
        proxy_pass http://127.0.0.1:5678;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_buffering off;
        proxy_cache off;
        chunked_transfer_encoding off;
    }
}
EOF

ssh root@<SERVER_IP> << 'EOF'
ln -sf /etc/nginx/sites-available/n8n /etc/nginx/sites-enabled/
rm -f /etc/nginx/sites-enabled/default
nginx -t && systemctl reload nginx
EOF
```

### Stap 3: SSL certificaat

```bash
ssh root@<SERVER_IP> "certbot --nginx -d <N8N_DOMAIN> --non-interactive --agree-tos -m <EMAIL>"
```

### Stap 4: Auto-renewal testen

```bash
ssh root@<SERVER_IP> "certbot renew --dry-run"
```

---

## Optie 2: Caddy (automatische SSL)

### Stap 1: Caddy installeren

```bash
ssh root@<SERVER_IP> << 'EOF'
apt install -y debian-keyring debian-archive-keyring apt-transport-https curl
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | tee /etc/apt/sources.list.d/caddy-stable.list
apt update && apt install -y caddy
EOF
```

### Stap 2: Caddy configuratie

```bash
ssh root@<SERVER_IP> "cat > /etc/caddy/Caddyfile" << 'EOF'
<N8N_DOMAIN> {
    reverse_proxy localhost:5678
}
EOF

ssh root@<SERVER_IP> "systemctl reload caddy"
```

Caddy regelt SSL automatisch via Let's Encrypt.

---

## Verificatie

```bash
# HTTPS werkt
curl -sI https://<N8N_DOMAIN>/ | head -5

# SSL certificaat geldig
echo | openssl s_client -connect <N8N_DOMAIN>:443 -servername <N8N_DOMAIN> 2>/dev/null | openssl x509 -noout -dates
```

Verwacht:
- HTTP 200 of 302 (redirect naar login)
- SSL certificaat geldig
