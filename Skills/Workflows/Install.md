# Install Workflow

Begeleide installatie van OpenClaw op een Hetzner Cloud VPS.

Begroet de gebruiker en kondig aan dat je begint:

```
We gaan OpenClaw installeren op Hetzner. Ik loop eerst even langs alles
wat we nodig hebben — wat ik zelf kan checken doe ik meteen, en als ik
jou ergens bij nodig heb, zeg ik het duidelijk. Klaar?
```

---

## Fase 0: Vereisten checken

Voer elke check uit en rapporteer de uitkomst aan de gebruiker.
Label elke stap duidelijk: **[Claude doet dit]** of **[Jij moet dit doen]**.

---

### Check 1 — hcloud CLI [Claude doet dit]

```bash
hcloud version
```

- **Aanwezig:** Noteer de versie. Ga door naar check 2.
- **Niet gevonden:** Zeg dit tegen de gebruiker:

  ```
  De hcloud CLI is nog niet geïnstalleerd — dat is het gereedschap waarmee
  we de Hetzner server aanmaken. Ik help je dat installeren.

  Op Mac (aanbevolen): voer dit uit in je terminal:
    brew install hcloud

  Op Linux:
    curl -fsSL https://raw.githubusercontent.com/hetznercloud/cli/main/install.sh | bash

  Laat me weten als dat klaar is, dan gaan we verder.
  ```
  Wacht op bevestiging. Controleer daarna opnieuw met `hcloud version`.

---

### Check 2 — Hetzner account & API-verbinding [Claude doet dit]

```bash
hcloud context list
```

- **Actieve context aanwezig:** Ga door naar check 3.
- **Leeg of geen actieve context:**

  ```
  We hebben een Hetzner Cloud account en een API-token nodig om servers
  te kunnen aanmaken.

  Heb je al een Hetzner Cloud account?
  ```

  **Als nog geen account:**
  ```
  Maak een account aan op hetzner.com/cloud — dat doe jij in de browser.
  Kom terug als het klaar is, dan gaan we verder met de API-token.
  ```

  **Als account wel aanwezig:**
  ```
  Ga naar: cloud.hetzner.com → jouw project → Security → API Tokens
  Maak een token aan met 'Read & Write' rechten.
  Kopieer het token (je ziet het maar één keer).

  Voer daarna dit uit in je terminal:
    hcloud context create openclaw

  Je wordt om het token gevraagd. Plak het in en druk op Enter.
  Laat me weten als dat gelukt is.
  ```
  Wacht op bevestiging. Controleer daarna met `hcloud context list`.

---

### Check 3 — SSH keypair [Claude doet dit]

```bash
ls ~/.ssh/id_ed25519.pub 2>/dev/null || ls ~/.ssh/id_rsa.pub 2>/dev/null && echo "KEYPAIR GEVONDEN" || echo "GEEN KEYPAIR"
```

- **Gevonden:** Noteer welke key aanwezig is. Ga door naar check 4.
- **Niet gevonden:**

  ```
  Er is nog geen SSH-sleutelpaar op je computer. Dat is de digitale sleutel
  waarmee je straks veilig kunt inloggen op je server — zonder wachtwoord.

  Ik maak hem aan voor je. Voer dit uit in je terminal:
    ssh-keygen -t ed25519 -C "openclaw" -f ~/.ssh/id_ed25519

  Je kunt op Enter drukken bij de vragen (geen wachtwoord op de key is
  prima voor servergebruik). Laat me weten als het klaar is.
  ```
  Wacht op bevestiging. Controleer daarna opnieuw.

---

### Check 4 — GitHub toegang OpenClaw [Claude doet dit]

```bash
curl -s -o /dev/null -w "%{http_code}" https://github.com/openclaw/openclaw
```

- **200:** Repository bereikbaar. Ga door naar check 5.
- **404 of fout:** De repository is niet bereikbaar of niet publiek. Informeer de gebruiker:
  ```
  De OpenClaw-repository is niet bereikbaar via jouw verbinding.
  Controleer of je toegang hebt tot github.com/openclaw/openclaw.
  Als de repo privé is en je toegang hebt: zorg dat je SSH-key geregistreerd
  staat bij GitHub voor je account.
  ```

---

### Check 5 — Telegram Bot Token [Jij moet dit doen]

```
Voor de installatie van OpenClaw heb je een Telegram Bot Token nodig.
Dat is de sleutel waarmee OpenClaw verbinding maakt met jouw Telegram-bot.

Heb je al een bot aangemaakt via @BotFather?
```

- **Ja:** Vraag: "Heb je het token bij de hand?" — noteer dat de gebruiker het later
  invult bij stap 4 van de installatie. Ga door.
- **Nee:**

  ```
  Doe het volgende in Telegram (dit duurt ongeveer 2 minuten):

  1. Open Telegram en zoek op @BotFather
  2. Stuur: /newbot
  3. Kies een naam voor je bot (bijv. "Mijn OpenClaw")
  4. Kies een gebruikersnaam (moet eindigen op 'bot', bijv. "mijnopenclaw_bot")
  5. BotFather stuurt je een token — dat ziet eruit als: 123456789:ABCdef...

  Bewaar dat token, je hebt het straks nodig. Geef een seintje als je klaar bent.
  ```
  Wacht op bevestiging.

---

### Samenvatting vereisten

Presenteer na alle checks een overzicht:

```
Vereisten-check klaar:
✓ hcloud CLI aanwezig (versie X.X.X)
✓ Hetzner account verbonden
✓ SSH keypair aanwezig
✓ GitHub toegang OK
✓ Telegram Bot Token beschikbaar

We kunnen beginnen. Ga je mee?
```

---

## Fase 1: Research

Voer eerst de research-stap uit. Lees en volg: `../Research.md`

Presenteer de uitkomst aan de gebruiker voordat je verdergaat.

---

## Fase 2: Keuzemenu

Stel de gebruiker de volgende vragen. Neem de antwoorden mee door de rest van de workflow.

### Vraag 1: Welke combinatie?

```
Wat wil je installeren?

A) Alleen een veilige Hetzner VPS (secure baseline)
B) VPS + OpenClaw (aanbevolen startpunt)
C) VPS + OpenClaw + Caddy (webserver erbij)
```

### Vraag 2: Tailscale?

```
Wil je Tailscale gebruiken voor veilige toegang? (aanbevolen)

Ja  → OpenClaw is alleen bereikbaar via jouw privé-netwerk. Veiliger en eenvoudiger.
Nee → We passen extra beveiligingsmaatregelen toe. Iets complexer.
```

### Bevestiging aan de gebruiker

Geef een overzicht van wat er geïnstalleerd wordt:

```
Installatie-plan:
✓ Hetzner VPS aanmaken (altijd)
✓ Dual-layer firewall instellen (altijd)
[als Tailscale=ja] ✓ Tailscale installeren
[als Tailscale=nee] ✓ Extra hardening zonder Tailscale
[als B of C] ✓ OpenClaw installeren
[als C] ✓ Caddy webserver installeren
✓ Verificatie-checklist nalopen

Doorgaan?
```

---

## Fase 3: Uitvoering

Voer de runbooks uit in de volgorde hieronder. Sla runbooks over die niet van toepassing
zijn op basis van de keuzes uit het keuzemenu.

### Stap 1 — VPS aanmaken (altijd)

Lees en volg: `../Runbooks/01-Vps.md`

### Stap 2 — Firewall instellen (altijd)

Lees en volg: `../Runbooks/02-Firewall.md`

### Stap 3a — Tailscale (alleen als Tailscale=ja)

Lees en volg: `../Runbooks/03a-Tailscale.md`

### Stap 3b — Extra hardening (alleen als Tailscale=nee)

Lees en volg: `../Runbooks/03b-Hardening.md`

> **Als combinatie C én Tailscale=nee:** Voer ook Sectie 5 van `05-Caddy.md` uit voor de
> publieke poortconfiguratie (Caddy moet op `0.0.0.0:443` luisteren i.p.v. `127.0.0.1:80`).

### Stap 4 — OpenClaw installeren (alleen als combinatie B of C)

Lees en volg: `../Runbooks/04-OpenClaw.md`

### Stap 5 — Caddy installeren (alleen als combinatie C)

Lees en volg: `../Runbooks/05-Caddy.md`

> **Als Tailscale=nee:** Volg ook Sectie 5 van dit runbook (publieke HTTPS-configuratie met
> domeinnaam + Let's Encrypt). Sla Sectie 5 niet over — zonder dit is Caddy verkeerd geconfigureerd.

---

## Fase 4: Verificatie

Lees en volg: `../Runbooks/06-Verify.md`

De verificatie-checklist past zich aan op de gemaakte keuzes. Loop alle punten na
en bevestig elk punt met de gebruiker.

---

## Afsluiting

Na succesvolle verificatie:

```
Installatie voltooid.

Samenvatting van wat er geïnstalleerd is:
- [componentenlijst op basis van keuzes]

Bewaar deze gegevens veilig:
- IP-adres van je VPS
- Gateway token (in /root/openclaw/.env)
- Tailscale-hostnaam (indien van toepassing)

Volgende stappen:
- Periodiek onderhoud: zeg "voer onderhoud uit op mijn OpenClaw VPS"
- Backup instellen: zit ingebouwd in de Maintain-workflow
```
