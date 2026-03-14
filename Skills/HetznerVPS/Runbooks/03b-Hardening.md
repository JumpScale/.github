# 03b-Hardening — Extra Beveiliging zonder Tailscale

## Context

Deze runbook is van toepassing als je bewust **geen Tailscale** gebruikt. Tailscale biedt een privé versleutelde tunnel tussen jouw apparaten en de VPS — zonder Tailscale is de OpenClaw gateway publiek bereikbaar via internet. Dat is niet per definitie onveilig, maar het vereist meer compenserende maatregelen.

Zonder Tailscale zijn de twee voornaamste risico's:
1. **Poort 443 staat publiek open** — meer aanvalsoppervlak dan met Tailscale (alleen poort 22).
2. **De gateway is direct bereikbaar** — een zwak token of Caddy-misconfiguratie leidt direct tot blootstelling.

Deze runbook beschrijft hoe je dat risico beheersbaar maakt.

---

## Sectie 1: Wat verandert er zonder Tailscale?

| Aspect | Met Tailscale | Zonder Tailscale |
|--------|--------------|-----------------|
| OpenClaw bereikbaar via | Privé Tailscale URL | Publiek IP + HTTPS |
| Poorten publiek open | Alleen 22 + 41641 | 22 + 443 |
| Risico | Laag | Hoger — meer aandacht nodig |
| Authenticatie gateway | Token via privé tunnel | Token over publiek internet (vereist TLS) |
| Fail2ban nodig | Optioneel | **Verplicht** |

---

## Sectie 2: HTTPS instellen (verplicht zonder Tailscale)

**Zonder Tailscale mag OpenClaw nooit bereikbaar zijn via plain HTTP.** TLS is geen luxe maar een vereiste: het gateway token reist anders in plaintext over het internet.

Wat je nodig hebt:
- Een domeinnaam die naar het publieke IP van de VPS wijst (A-record)
- Caddy geconfigureerd met dat domein — Caddy regelt automatisch Let's Encrypt certificaatuitgifte en -verlenging
- Een sterk gateway token (zie Sectie 3)

**Verschil in Caddy-configuratie:**

Zie `05-Caddy.md` voor de volledige Caddy-setup. Het kritieke verschil zonder Tailscale:

| Instelling | Met Tailscale | Zonder Tailscale |
|-----------|--------------|-----------------|
| Caddy listen | `127.0.0.1:80` | `0.0.0.0:443` (of domeinnaam) |
| TLS | Optioneel (tunnel is al versleuteld) | Verplicht — Caddy doet dit automatisch met Let's Encrypt |
| Domeinnaam | Niet vereist | Vereist voor Let's Encrypt |

Caddy haalt automatisch een Let's Encrypt certificaat op als je een domeinnaam configureert. Zorg dat de DNS A-record klopt vóór je Caddy start.

**Extra Hetzner-firewallregel (alleen zonder Tailscale):**

```bash
# HTTPS toestaan — alleen toevoegen als er geen Tailscale wordt gebruikt
hcloud firewall add-rule <FIREWALL-NAAM> \
  --direction in --protocol tcp --port 443 \
  --source-ips 0.0.0.0/0 --source-ips ::/0
```

Zonder deze regel kan Caddy geen verbindingen accepteren van buitenaf, en kan Let's Encrypt ook geen HTTP-01 challenge voltooien.

---

## Sectie 3: Gateway token versterken

Het gateway token is de enige authenticatielaag voor de OpenClaw API. Zonder Tailscale is dit des te kritischer.

```bash
# Genereer een cryptografisch sterk token (256 bits entropie)
openssl rand -hex 32
```

Sla het gegenereerde token op in `/root/openclaw/.env` als de waarde van `OPENCLAW_GATEWAY_TOKEN`. Het auto-gegenereerde token bij installatie is al voldoende sterk — gebruik dit commando als je het wil vervangen of roteren.

**Aandachtspunten:**
- Deel het token nooit via onversleutelde kanalen (email, Slack zonder encryptie).
- Roteer het token als je vermoedt dat het gecompromitteerd is — dit vereist een herstart van de gateway container.
- Bewaar het token op een veilige plek (password manager).

---

## Sectie 4: IP-restrictie via UFW (optioneel maar aanbevolen)

Als je alleen verbinding maakt vanuit bekende IP-ranges (kantoor, thuis), kun je poort 443 beperken tot die IP's. Dit elimineert de meeste scanners en bots.

```bash
# Stap 1: toegang vanuit jouw specifieke IP toestaan
ssh root@<IP> "ufw allow from <JOUW-IP> to any port 443"

# Stap 2: de open regel (0.0.0.0/0) verwijderen
ssh root@<IP> "ufw delete allow 443/tcp"

# Controleer het resultaat
ssh root@<IP> "ufw status verbose"
```

> Doe stap 1 altijd vóór stap 2 — anders sluit je jezelf buitenaf.

Als je IP-adres dynamisch is (wisselend), is deze aanpak onpraktisch. In dat geval is fail2ban (Sectie 5) een betere compenserende maatregel.

---

## Sectie 5: Fail2ban instellen (verplicht zonder Tailscale)

Zonder Tailscale zijn je services direct blootgesteld aan internet-scanners en bruteforce-pogingen. Fail2ban is **verplicht**: het detecteert herhaalde mislukte pogingen en blokkeert automatisch de bron-IP. Sla deze stap niet over.

```bash
ssh root@<IP> "apt-get install -y fail2ban && systemctl enable --now fail2ban"
```

Fail2ban beschermt standaard SSH. Voeg ook een jail toe voor Caddy-toegang (HTTP 401-responses):

```bash
ssh root@<IP> "cat > /etc/fail2ban/jail.d/caddy.conf" << 'EOF'
[caddy-auth]
enabled  = true
port     = http,https
filter   = caddy-auth
logpath  = /var/log/caddy/access.log
maxretry = 5
bantime  = 3600
EOF
ssh root@<IP> "systemctl reload fail2ban"
```

Controleer de status:
```bash
ssh root@<IP> "fail2ban-client status"
ssh root@<IP> "fail2ban-client status sshd"
```

---

## Sectie 6: Aandachtspunten

Zonder Tailscale zijn dit de punten die extra aandacht vragen:

- **Gateway token**: houd het geheim, sla het veilig op in een password manager, en roteer het periodiek of bij twijfel.
- **Logs monitoren**: controleer regelmatig op ongeautoriseerde toegangspogingen via `journalctl -u caddy` en `fail2ban-client status`.
- **IP-allowlisting**: als je IP-adres stabiel is, is IP-restrictie via UFW de meest effectieve extra beveiligingsmaatregel.
- **Certificaatverlening**: Caddy verlengt Let's Encrypt certificaten automatisch, maar controleer af en toe of dit succesvol verloopt — een verlopen certificaat blokkeert alle toegang.
- **Poort 443 in Hetzner-firewall**: als je de service tijdelijk wil afsluiten voor buitenstaanders, verwijder dan de Hetzner-firewallregel voor poort 443 — dat werkt sneller dan UFW-wijzigingen.
