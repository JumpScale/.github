# Stap 6: Security hardening

## Doel
Alle publiek bereikbare services beveiligen met best practices.

---

## 6.1 Nginx server tokens verbergen [Claude doet dit]

In `/etc/nginx/nginx.conf`, uncomment of voeg toe:
```
server_tokens off;
```

## 6.2 TLS versies beperken [Claude doet dit]

In `/etc/nginx/nginx.conf`, vervang de ssl_protocols regel:
```
ssl_protocols TLSv1.2 TLSv1.3;
```

Verwijder TLSv1 en TLSv1.1 als deze nog aanwezig zijn.

## 6.3 Security headers toevoegen [Claude doet dit]

```bash
cat > /etc/nginx/conf.d/security-headers.conf << 'EOF'
# Security Headers
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-Content-Type-Options "nosniff" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
add_header Permissions-Policy "camera=(), microphone=(), geolocation=()" always;
EOF
```

## 6.4 UFW opschonen [Claude doet dit]

Controleer welke poorten open zijn:
```bash
ufw status numbered
```

Verwijder onnodige regels. Alleen deze poorten zijn nodig:
- 22/tcp (SSH)
- 80/tcp (HTTP redirect)
- 443/tcp (HTTPS)

```bash
# Voorbeeld: verwijder onnodige poort
ufw delete allow 3000/tcp
```

## 6.5 Nginx herladen [Claude doet dit]

```bash
nginx -t && systemctl reload nginx
```

---

## Verificatie

```bash
# Server versie verborgen
curl -sI https://<DOMEIN> | grep -i server
# Verwacht: "Server: nginx" (zonder versienummer)

# Security headers aanwezig
curl -sI https://<DOMEIN> | grep -iE "x-frame|strict-transport|x-content|referrer|permissions"
# Verwacht: alle 5 headers aanwezig

# Alleen noodzakelijke poorten open
ufw status
```
