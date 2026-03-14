# Install Workflow

Begeleide installatie van OpenClaw op een bestaande server.

Begroet de gebruiker en kondig aan dat je begint:

```
We gaan OpenClaw installeren op je server. Ik loop eerst even langs alles
wat we nodig hebben  -  wat ik zelf kan checken doe ik meteen, en als ik
jou ergens bij nodig heb, zeg ik het duidelijk. Klaar?
```

---

## Fase 0: Vereisten checken

Voer elke check uit en rapporteer de uitkomst aan de gebruiker.
Label elke stap duidelijk: **[Claude doet dit]** of **[Jij moet dit doen]**.

---

### Check 1  -  SSH toegang [Claude doet dit]

Vraag de gebruiker om het IP-adres of de hostnaam van de server.

```bash
ssh root@<IP> "echo 'SSH OK' && uname -a"
```

- **Succesvol:** Noteer het OS. Ga door.
- **Mislukt:** Help de gebruiker met de SSH-verbinding. Als er nog geen server is,
  verwijs naar de **[HetznerVPS skill](../../HetznerVPS/)**.

---

### Check 2  -  Docker [Claude doet dit]

```bash
ssh root@<IP> "docker --version && docker compose version"
```

- **Aanwezig:** Ga door.
- **Niet gevonden:** Dit wordt opgelost in runbook [01-Docker.md](../Runbooks/01-Docker.md).

---

### Check 3  -  GitHub toegang OpenClaw [Claude doet dit]

```bash
curl -s -o /dev/null -w "%{http_code}" https://github.com/openclaw/openclaw
```

- **200:** Repository bereikbaar.
- **404 of fout:**
  ```
  De OpenClaw-repository is niet bereikbaar. Controleer of je toegang hebt
  tot github.com/openclaw/openclaw.
  ```

---

### Check 4  -  Telegram Bot Token [Jij moet dit doen]

```
Voor de installatie van OpenClaw heb je een Telegram Bot Token nodig.
Dat is de sleutel waarmee OpenClaw verbinding maakt met jouw Telegram-bot.

Heb je al een bot aangemaakt via @BotFather?
```

- **Ja:** Vraag of het token bij de hand is.
- **Nee:**

  ```
  Doe het volgende in Telegram (dit duurt ongeveer 2 minuten):

  1. Open Telegram en zoek op @BotFather
  2. Stuur: /newbot
  3. Kies een naam voor je bot (bijv. "Mijn OpenClaw")
  4. Kies een gebruikersnaam (moet eindigen op 'bot', bijv. "mijnopenclaw_bot")
  5. BotFather stuurt je een token  -  dat ziet eruit als: 123456789:ABCdef...

  Bewaar dat token, je hebt het straks nodig.
  ```

---

### Samenvatting vereisten

```
Vereisten-check klaar:
✓ SSH toegang tot server
✓ Docker [aanwezig/wordt geïnstalleerd]
✓ GitHub toegang OK
✓ Telegram Bot Token beschikbaar

We kunnen beginnen.
```

---

## Fase 1: Research

Voer eerst de research-stap uit. Lees en volg: `../Research.md`

Presenteer de uitkomst aan de gebruiker voordat je verdergaat.

---

## Fase 2: Keuzemenu

### Vraag: Caddy webserver?

```
Wil je ook een Caddy webserver erbij? (optioneel)

Ja  → Je kunt bestanden delen via een beveiligde URL (exports, rapporten, documenten)
Nee → Alleen de OpenClaw gateway. Je kunt Caddy later altijd nog toevoegen.
```

### Bevestiging

```
Installatie-plan:
✓ Docker installeren (indien nodig)
✓ OpenClaw gateway installeren en configureren
✓ Telegram-bot koppelen
[als Caddy=ja] ✓ Caddy webserver installeren

Doorgaan?
```

---

## Fase 3: Uitvoering

### Stap 1  -  Docker (indien nodig)

Lees en volg: `../Runbooks/01-Docker.md`

### Stap 2  -  OpenClaw installeren

Lees en volg: `../Runbooks/02-OpenClaw.md`

### Stap 3  -  Caddy (alleen als gekozen)

Lees en volg: `../Runbooks/03-Caddy.md`

---

## Fase 4: Verificatie

Lees en volg: `../Runbooks/04-Verify.md`

---

## Afsluiting

```
OpenClaw installatie voltooid.

Samenvatting:
- [componentenlijst op basis van keuzes]

Bewaar deze gegevens veilig:
- Gateway token (in /root/openclaw/.env)
- Telegram bot naam en token

Volgende stappen:
- Periodiek onderhoud: zeg "voer onderhoud uit op mijn OpenClaw"
- Backup instellen: zit ingebouwd in de Maintain-workflow
```
