# Stap 3: Reverse Proxy + SSL

## Doel
HTTPS-toegang tot Twenty CRM via een domeinnaam met automatisch SSL-certificaat.

---

## Optie 1: nginx (standaard)

### 3.1 nginx installeren [Claude doet dit]

```bash
apt install -y nginx certbot python3-certbot-nginx
```

### 3.2 Site configuratie [Claude doet dit]

```bash
cat > /etc/nginx/sites-available/twenty-crm << 'EOF'
server {
    listen 80;
    server_name <DOMEIN>;

    client_max_body_size 64M;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_buffering off;
        proxy_request_buffering off;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
EOF

ln -sf /etc/nginx/sites-available/twenty-crm /etc/nginx/sites-enabled/
nginx -t && systemctl reload nginx
```

### 3.3 SSL certificaat [Claude doet dit]

Controleer eerst of DNS naar het juiste IP wijst:

```bash
dig +short <DOMEIN>
```

Als het IP klopt:
```bash
certbot --nginx -d <DOMEIN> --non-interactive --agree-tos -m <EMAIL>
```

---

## Optie 2: Caddy (alternatief)

### 3.1 Caddy installeren [Claude doet dit]

```bash
apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | tee /etc/apt/sources.list.d/caddy-stable.list
apt update && apt install -y caddy
```

### 3.2 Caddyfile [Claude doet dit]

```bash
cat > /etc/caddy/Caddyfile << 'EOF'
<DOMEIN> {
    reverse_proxy 127.0.0.1:3000
}
EOF
systemctl restart caddy
```

Caddy regelt SSL automatisch.

---

## Verificatie

```bash
curl -sI https://<DOMEIN> | head -5    # HTTP/1.1 200 OK
curl -sI https://<DOMEIN> | grep -i strict-transport  # HSTS header aanwezig (na security stap)
```
