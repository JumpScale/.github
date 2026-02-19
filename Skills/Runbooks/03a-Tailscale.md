# 03a-Tailscale — Veilige Toegang via Tailscale

## Wat is Tailscale?

Tailscale is een VPN-dienst die een privaat netwerk aanmaakt tussen jouw apparaten. Zodra Tailscale op de VPS is geinstalleerd, is de server bereikbaar via dit private netwerk — niet via het publieke internet. OpenClaw's interface hoeft daardoor niet publiek blootgesteld te worden.

**Voordelen:**
- Geen publieke HTTPS-poort nodig voor OpenClaw
- Toegang vanaf elk apparaat waarop Tailscale is geinstalleerd
- Automatisch certificaatbeheer via Tailscale HTTPS (geen eigen domein nodig)
- Toegang eenvoudig intrekken door een apparaat te verwijderen uit de Tailscale console

---

## Sectie 1: Tailscale installeren op de VPS

Het installatiescript detecteert automatisch de Linux-distributie en installeert de juiste pakketversie:

```bash
ssh root@<IP> << 'EOF'
curl -fsSL https://tailscale.com/install.sh | sh
EOF
```

---

## Sectie 2: VPS koppelen aan je Tailscale-netwerk

Koppel de VPS aan jouw Tailscale-account. De hostnaam bepaalt hoe de server zichtbaar is in je netwerk:

```bash
ssh root@<IP> "tailscale up --hostname <SERVERNAAM>"
```

Het commando geeft een URL terug — open deze in je browser om de VPS te autoriseren in de Tailscale console.

Na autorisatie, verifieer de verbinding:

```bash
ssh root@<IP> "tailscale status"
# Je ziet nu een Tailscale IP (100.x.x.x) en de hostnaam
```

---

## Sectie 3: Tailscale HTTPS inschakelen

Tailscale kan automatisch HTTPS-certificaten verzorgen voor je VPS — zonder eigen domein. Dit werkt via `tailscale serve`, dat de service beschikbaar maakt op het Tailscale-netwerk:

```bash
# Als OpenClaw direct op poort 18789 luistert:
ssh root@<IP> "tailscale serve --https=443 http://localhost:18789"

# Als Caddy gebruikt wordt als tussenlaag:
ssh root@<IP> "tailscale serve --https=443 http://localhost:80"
```

Na activatie is OpenClaw bereikbaar op:
`https://<SERVERNAAM>.<TAILNET-NAAM>.ts.net`

Controleer de actieve serve-configuratie:

```bash
ssh root@<IP> "tailscale serve status"
```

---

## Sectie 4: Toegang instellen op andere apparaten

Installeer Tailscale op je Mac, Windows-machine of telefoon via [tailscale.com/download](https://tailscale.com/download).

Zodra je apparaat verbonden is met hetzelfde Tailscale-netwerk, is OpenClaw bereikbaar op:

```
https://<TAILSCALE-HOSTNAME>/
```

Geen extra configuratie nodig — Tailscale regelt de routering en het certificaat automatisch.

---

## Sectie 5: UFW openen voor Tailscale

Nu Tailscale geïnstalleerd en verbonden is, voeg je de UFW-regel toe die al het verkeer via
de Tailscale-interface toestaat. Tailscale regelt zelf authenticatie en encryptie:

```bash
ssh root@<IP> << 'EOF'
ufw allow in on tailscale0
ufw status verbose
EOF
```

Verwachte output bevat:
```
Anywhere on tailscale0    ALLOW IN    Anywhere
```

> Deze regel is bewust hier toegevoegd (en niet in `02-Firewall.md`): zo staat de regel
> alleen in UFW als Tailscale ook daadwerkelijk geïnstalleerd en actief is.

---

## Sectie 6: Directe verbindingen (DERP vs. direct)

Tailscale probeert altijd een directe peer-to-peer verbinding op te zetten via UDP poort 41641. Dit is sneller dan een relay. Als een directe verbinding niet mogelijk is (bijv. door NAT), valt Tailscale terug op DERP-relayservers (Tailscale's eigen infrastructuur).

De firewallregel voor UDP 41641 (ingesteld in `02-Firewall.md`) maakt directe verbindingen mogelijk. Zonder deze regel werkt Tailscale nog steeds, maar via relay — met iets hogere latency.

Controleer welk verbindingstype actief is:

```bash
ssh root@<IP> "tailscale status"
# Zoek naar: "relay" (DERP) of "direct" in de output
```

Een directe verbinding ziet er zo uit:
```
100.x.x.x   jouw-laptop   owner  linux   active; direct 1.2.3.4:41641
```

Bij een relay:
```
100.x.x.x   jouw-laptop   owner  linux   active; relay "ams"
```
