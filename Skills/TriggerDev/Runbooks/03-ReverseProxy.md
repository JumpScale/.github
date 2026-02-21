# 03 -- Reverse Proxy + SSL

## Doel

Trigger.dev bereikbaar via HTTPS met automatisch SSL-certificaat.

---

## Stap 1: nginx installeren

```bash
ssh root@<SERVER_IP> << 'EOF'
apt install -y nginx certbot python3-certbot-nginx
systemctl enable nginx
EOF
```

## Stap 2: nginx configuratie

```bash
ssh root@<SERVER_IP> "cat > /etc/nginx/sites-available/trigger" << 'EOF'
server {
    listen 80;
    server_name <TRIGGER_DOMAIN>;

    location / {
        proxy_pass http://127.0.0.1:3040;
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
ln -sf /etc/nginx/sites-available/trigger /etc/nginx/sites-enabled/
rm -f /etc/nginx/sites-enabled/default
nginx -t && systemctl reload nginx
EOF
```

## Stap 3: SSL certificaat

```bash
ssh root@<SERVER_IP> "certbot --nginx -d <TRIGGER_DOMAIN> --non-interactive --agree-tos -m <EMAIL>"
```

## Stap 4: Auto-renewal testen

```bash
ssh root@<SERVER_IP> "certbot renew --dry-run"
```

---

## Verificatie

```bash
# HTTPS werkt
curl -sI https://<TRIGGER_DOMAIN>/ | head -5

# SSL certificaat geldig
echo | openssl s_client -connect <TRIGGER_DOMAIN>:443 -servername <TRIGGER_DOMAIN> 2>/dev/null | openssl x509 -noout -dates
```

Verwacht:
- HTTP 200 of 302 (redirect naar login)
- SSL certificaat geldig
