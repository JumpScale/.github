# Install Workflow

Begeleide installatie van een veilige Hetzner Cloud VPS.

Begroet de gebruiker en kondig aan dat je begint:

```
We gaan een veilige VPS opzetten op Hetzner. Ik loop eerst even langs alles
wat we nodig hebben  -  wat ik zelf kan checken doe ik meteen, en als ik
jou ergens bij nodig heb, zeg ik het duidelijk. Klaar?
```

---

## Fase 0: Vereisten checken

Voer elke check uit en rapporteer de uitkomst aan de gebruiker.
Label elke stap duidelijk: **[Claude doet dit]** of **[Jij moet dit doen]**.

---

### Check 1  -  hcloud CLI [Claude doet dit]

```bash
hcloud version
```

- **Aanwezig:** Noteer de versie. Ga door naar check 2.
- **Niet gevonden:** Zeg dit tegen de gebruiker:

  ```
  De hcloud CLI is nog niet geïnstalleerd  -  dat is het gereedschap waarmee
  we de Hetzner server aanmaken. Ik help je dat installeren.

  Op Mac (aanbevolen): voer dit uit in je terminal:
    brew install hcloud

  Op Linux:
    curl -fsSL https://raw.githubusercontent.com/hetznercloud/cli/main/install.sh | bash

  Laat me weten als dat klaar is, dan gaan we verder.
  ```
  Wacht op bevestiging. Controleer daarna opnieuw met `hcloud version`.

---

### Check 2  -  Hetzner account & API-verbinding [Claude doet dit]

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
  Maak een account aan op hetzner.com/cloud  -  dat doe jij in de browser.
  Kom terug als het klaar is, dan gaan we verder met de API-token.
  ```

  **Als account wel aanwezig:**
  ```
  Ga naar: cloud.hetzner.com → jouw project → Security → API Tokens
  Maak een token aan met 'Read & Write' rechten.
  Kopieer het token (je ziet het maar één keer).

  Voer daarna dit uit in je terminal:
    hcloud context create mijn-server

  Je wordt om het token gevraagd. Plak het in en druk op Enter.
  Laat me weten als dat gelukt is.
  ```
  Wacht op bevestiging. Controleer daarna met `hcloud context list`.

---

### Check 3  -  SSH keypair [Claude doet dit]

```bash
ls ~/.ssh/id_ed25519.pub 2>/dev/null || ls ~/.ssh/id_rsa.pub 2>/dev/null && echo "KEYPAIR GEVONDEN" || echo "GEEN KEYPAIR"
```

- **Gevonden:** Noteer welke key aanwezig is. Ga door.
- **Niet gevonden:**

  ```
  Er is nog geen SSH-sleutelpaar op je computer. Dat is de digitale sleutel
  waarmee je straks veilig kunt inloggen op je server  -  zonder wachtwoord.

  Voer dit uit in je terminal:
    ssh-keygen -t ed25519 -C "mijn-server" -f ~/.ssh/id_ed25519

  Je kunt op Enter drukken bij de vragen (geen wachtwoord op de key is
  prima voor servergebruik). Laat me weten als het klaar is.
  ```
  Wacht op bevestiging. Controleer daarna opnieuw.

---

### Samenvatting vereisten

Presenteer na alle checks een overzicht:

```
Vereisten-check klaar:
✓ hcloud CLI aanwezig (versie X.X.X)
✓ Hetzner account verbonden
✓ SSH keypair aanwezig

We kunnen beginnen. Ga je mee?
```

---

## Fase 1: Research

Voer eerst de research-stap uit. Lees en volg: `../Research.md`

Presenteer de uitkomst aan de gebruiker voordat je verdergaat.

---

## Fase 2: Keuzemenu

### Vraag: Tailscale?

```
Wil je Tailscale gebruiken voor veilige toegang? (aanbevolen)

Ja  → Je server is alleen bereikbaar via jouw privé-netwerk. Veiliger en eenvoudiger.
Nee → We passen extra beveiligingsmaatregelen toe (fail2ban, IP-restrictie). Iets complexer.
```

### Bevestiging

```
Installatie-plan:
✓ Hetzner VPS aanmaken
✓ Dual-layer firewall instellen
[als Tailscale=ja] ✓ Tailscale installeren
[als Tailscale=nee] ✓ Extra hardening

Doorgaan?
```

---

## Fase 3: Uitvoering

Voer de runbooks uit in volgorde. Sla over wat niet van toepassing is.

### Stap 1  -  VPS aanmaken (altijd)

Lees en volg: `../Runbooks/01-Vps.md`

### Stap 2  -  Firewall instellen (altijd)

Lees en volg: `../Runbooks/02-Firewall.md`

### Stap 3a  -  Tailscale (alleen als Tailscale=ja)

Lees en volg: `../Runbooks/03a-Tailscale.md`

### Stap 3b  -  Extra hardening (alleen als Tailscale=nee)

Lees en volg: `../Runbooks/03b-Hardening.md`

---

## Fase 4: Verificatie

Lees en volg: `../Runbooks/04-Verify.md`

---

## Afsluiting

Na succesvolle verificatie:

```
VPS-installatie voltooid.

Samenvatting:
- [componentenlijst op basis van keuzes]

Bewaar deze gegevens veilig:
- IP-adres van je VPS
- Tailscale-hostnaam (indien van toepassing)

Volgende stappen:
- Applicaties installeren (bijv. OpenClaw, Twenty CRM, n8n)
- Periodiek onderhoud: zeg "controleer de security van mijn VPS"
```
