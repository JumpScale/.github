# 05-Caddy — Webserver voor Bestanden

## Wat is Caddy?

Caddy is een moderne webserver die HTTPS automatisch afhandelt — inclusief certificaataanvraag en -verlenging. In de OpenClaw-setup heeft Caddy twee rollen:

1. **Reverse proxy**: verkeer doorsturen naar de OpenClaw gateway (poort 18789)
2. **File server**: bestanden in een aangewezen map beschikbaar maken via een URL

Dit is handig voor het delen van documenten, exports of andere bestanden die OpenClaw aanmaakt. Caddy draait als Docker-container naast OpenClaw.

---

## Sectie 1: Caddy in docker-compose.yml

Caddy is onderdeel van de `docker-compose.yml` die aangemaakt is tijdens de OpenClaw-installatie. Het relevante blok:

```yaml
caddy:
  image: caddy:2-alpine
  container_name: openclaw-caddy
  ports:
    - "127.0.0.1:80:80"   # Alleen localhost — Tailscale zorgt voor extern bereik
  volumes:
    - ./Caddyfile:/etc/caddy/Caddyfile:ro
    - /root/clawd/public:/srv/web:ro    # Bestanden hier worden geserveerd
    - caddy_data:/data
    - caddy_config:/config
  networks:
    - openclaw-net
  restart: unless-stopped
  deploy:
    resources:
      limits:
        memory: 256m
        cpus: "0.5"
```

**Toelichting volumes:**
- `./Caddyfile` — de routeringsconfiguratie (zie Sectie 2)
- `/root/clawd/public:/srv/web` — bestanden die je hier plaatst worden geserveerd via `/web/`; de container ziet ze als `/srv/web`
- `caddy_data` en `caddy_config` — persistente opslag voor certificaten en interne Caddy-state

De poortbinding `127.0.0.1:80:80` zorgt dat Caddy niet direct van buiten bereikbaar is. Tailscale (of een eigen domein) regelt de externe toegang.

---

## Sectie 2: Caddyfile aanmaken

De Caddyfile bepaalt hoe Caddy inkomende verzoeken verdeelt:

```bash
ssh root@<IP> "cat > /root/openclaw/Caddyfile" << 'EOF'
:80 {
    # Statische bestanden serveren
    handle /web/* {
        uri strip_prefix /web
        file_server {
            root /srv/web
        }
    }

    # Alle andere requests naar OpenClaw gateway
    reverse_proxy openclaw-gateway:18789
}
EOF
```

**Hoe de routering werkt:**
- Verzoeken naar `/web/*` worden afgehandeld als statische bestanden uit `/root/clawd/public/`
- Alle overige verzoeken worden doorgezet naar de OpenClaw gateway op poort 18789
- `uri strip_prefix /web` verwijdert het `/web`-prefix voordat het pad opgezocht wordt in de root-map — zo wordt `/web/rapport.pdf` vertaald naar `/srv/web/rapport.pdf`

---

## Sectie 3: Publieke map aanmaken

Maak de map aan waar OpenClaw (en jijzelf) bestanden in kan plaatsen:

```bash
ssh root@<IP> "mkdir -p /root/clawd/public"
```

Bestanden die je in `/root/clawd/public/` plaatst zijn automatisch bereikbaar via:

- **Met Tailscale:** `https://<TAILSCALE-HOSTNAME>/web/`
- **Zonder Tailscale:** `https://<DOMEIN>/web/`

De map is gemount als read-only in de container (`:ro`), zodat Caddy geen schrijfrechten heeft op je bestanden.

---

## Sectie 4: Caddy starten

```bash
ssh root@<IP> "cd /root/openclaw && docker compose up -d caddy"
ssh root@<IP> "docker logs openclaw-caddy --tail=20"
```

Caddy start normaal gesproken in minder dan een seconde. De logs tonen of de Caddyfile correct is geladen en of er verbindingsfouten zijn naar de gateway.

---

## Sectie 5: Zonder Tailscale — extra configuratie

Als Tailscale niet gebruikt wordt, moet Caddy zelf de publieke HTTPS-verbinding afhandelen. Caddy vraagt dan automatisch een Let's Encrypt-certificaat aan. Pas de Caddyfile aan:

```
# Vervang ":80" met je domeinnaam — Caddy regelt het certificaat automatisch
jouwdomein.nl {
    handle /web/* {
        uri strip_prefix /web
        file_server {
            root /srv/web
        }
    }
    reverse_proxy openclaw-gateway:18789
}
```

En stel de poortbinding in `docker-compose.yml` in om ook van buiten bereikbaar te zijn:

```yaml
ports:
  - "80:80"    # Nodig voor Let's Encrypt HTTP-verificatie
  - "443:443"  # HTTPS
```

Let op: Let's Encrypt vereist dat het domein via DNS naar het publieke IP van de VPS wijst. Caddy handelt de rest automatisch af — geen certbot of cron-job nodig.

---

## Sectie 6: Verificatie

```bash
# Container draait?
ssh root@<IP> "docker ps | grep caddy"

# Logs schoon?
ssh root@<IP> "docker logs openclaw-caddy --tail=10"

# Bereikbaar via Tailscale?
curl -s -o /dev/null -w "%{http_code}" https://<TAILSCALE-HOSTNAME>/web/
# Verwacht: 200 (als er bestanden zijn) of 404 (map leeg — dat is ok)
```

Een lege map geeft een 404 terug, maar dat is het verwachte gedrag van de file server. Controleer of de routing werkt door een testbestand te plaatsen:

```bash
ssh root@<IP> "echo 'test' > /root/clawd/public/test.txt"
curl https://<TAILSCALE-HOSTNAME>/web/test.txt
# Verwacht: "test"
ssh root@<IP> "rm /root/clawd/public/test.txt"
```
