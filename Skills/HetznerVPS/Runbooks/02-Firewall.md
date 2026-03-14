# 02-Firewall — Dual-Layer Beveiliging

## Architectuur

Deze runbook configureert twee onafhankelijke firewalllagen. Het principe: als één laag verkeerd geconfigureerd is, vangt de andere het op.

```
Internet
   │
   ▼
[Hetzner Cloud Firewall]  ← Laag 1: blokkeert vóór de VM
   │  TCP 22, UDP 41641, ICMP doorgelaten
   ▼
[VPS: UFW]               ← Laag 2: OS-kernel
   │  SSH rate-limit, tailscale0 ALLOW
   ▼
[Services]
```

**Waarom twee lagen?**

- **Hetzner Cloud Firewall** werkt op netwerkniveau, vóór het verkeer de VM bereikt. Dit beschermt ook tijdens `ufw disable`, kernel panics, of andere OS-problemen.
- **UFW** werkt in de OS-kernel en biedt fijnmazigere controle zoals rate-limiting en interface-specifieke regels.
- Als je per ongeluk UFW uitschakelt, staat de Hetzner-firewall nog steeds aan — en andersom.

---

## Sectie 1: Hetzner Cloud Firewall (Laag 1)

### Stap 1: Firewall aanmaken

```bash
hcloud firewall create --name <FIREWALL-NAAM>
```

Gebruik als naam de conventie `<SERVERNAAM>-fw` zodat de koppeling duidelijk is, bijvoorbeeld `openclaw-vps-fw`.

### Stap 2: Inkomende regels toevoegen

Voeg per regel toe en begrijp waarom elk poort nodig is:

```bash
# TCP 22: SSH toegang — zonder dit kun je niet inloggen
hcloud firewall add-rule <FIREWALL-NAAM> \
  --direction in --protocol tcp --port 22 \
  --source-ips 0.0.0.0/0 --source-ips ::/0

# UDP 41641: Tailscale directe verbindingen (DERP relay-fallback gebruikt andere paden)
hcloud firewall add-rule <FIREWALL-NAAM> \
  --direction in --protocol udp --port 41641 \
  --source-ips 0.0.0.0/0 --source-ips ::/0

# ICMP: ping en netwerk-diagnostiek — helpt bij troubleshooting
hcloud firewall add-rule <FIREWALL-NAAM> \
  --direction in --protocol icmp \
  --source-ips 0.0.0.0/0 --source-ips ::/0
```

> **Let op: outbound verkeer** is standaard ALLOW in Hetzner Cloud Firewall. Dit is bewust zo gelaten: de services op de VPS (OpenClaw gateway, Telegram bot, Anthropic API calls) hebben uitgaande verbindingen nodig.

### Stap 3: Firewall koppelen aan de server

```bash
hcloud firewall apply-to-resource \
  --type server \
  --server <SERVERNAAM> \
  <FIREWALL-NAAM>
```

### Stap 4: Verifiëren

```bash
hcloud firewall describe <FIREWALL-NAAM>
# Verwacht output:
# - 3 inkomende regels (TCP 22, UDP 41641, ICMP)
# - Applied to: server <SERVERNAAM>
```

### Stap 5: SSH testen

Controleer direct na activatie of SSH nog werkt:

```bash
ssh root@<IP> "echo 'Firewall OK'"
```

Als dit mislukt: gebruik de **Hetzner Console** (webterm via Cloud Console) om de regels te controleren.

---

## Sectie 2: UFW (Laag 2)

Voer het volgende uit als een enkel SSH-blok om te voorkomen dat je jezelf buitensluit:

```bash
ssh root@<IP> << 'EOF'
ufw default deny incoming
ufw default allow outgoing
ufw limit 22/tcp comment 'SSH ratelimit'
ufw --force enable
ufw status verbose
EOF
```

**Toelichting per regel:**

- `default deny incoming`: alles wat niet expliciet is toegestaan, wordt geblokkeerd.
- `default allow outgoing`: services op de VPS mogen verbinding maken naar buiten.
- `ufw limit 22/tcp`: SSH rate-limiting — maximaal 6 verbindingspogingen per 30 seconden. Dit stopt bruteforce-aanvallen automatisch.
- `--force enable`: schakelt UFW in zonder interactieve bevestiging (nodig voor scripts).

> **Tailscale-regel:** De `ufw allow in on tailscale0`-regel wordt toegevoegd in `03a-Tailscale.md`,
> nadat Tailscale geïnstalleerd is. Zo staat de regel er alleen als Tailscale ook echt actief is.

---

## Sectie 3: Verificatie

Controleer beide lagen:

```bash
# Hetzner firewall-configuratie
hcloud firewall describe <FIREWALL-NAAM>

# UFW-status op de VPS
ssh root@<IP> "ufw status verbose"
```

Verwachte UFW-output:
```
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)

To                         Action      From
--                         ------      ----
22/tcp                     LIMIT IN    Anywhere        # SSH ratelimit
```

(De Tailscale-regel verschijnt hier pas na `03a-Tailscale.md`.)

---

## Sectie 4: Incident Response

**VPS niet bereikbaar via SSH:**
Gebruik de **Hetzner Console** (Cloud Console → server → Console-tab) — dit is een webgebaseerde terminal die altijd beschikbaar is, ongeacht firewallstatus.

**UFW heeft SSH per ongeluk geblokkeerd:**
Herstel via de Hetzner Console-webterm:
```bash
ufw allow 22/tcp && ufw reload
```

**Hetzner-firewall per ongeluk verwijderd:**
Herhaal Sectie 1 volledig. Alle regels zijn idempotent — je kunt ze opnieuw toevoegen zonder problemen.

**Tailscale werkt niet meer:**
Controleer of UDP 41641 nog open staat in de Hetzner-firewall:
```bash
hcloud firewall describe <FIREWALL-NAAM>
```
