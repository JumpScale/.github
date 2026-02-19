# 01 — VPS Aanmaken en Basisconfiguratie

Runbook voor het opzetten van een nieuwe Hetzner Cloud VPS met SSH hardening en automatische beveiligingsupdates. Dit is de eerste stap voordat je Tailscale, Caddy of applicaties installeert.

---

## Vereisten vooraf

Zorg dat het volgende aanwezig is voordat je begint:

- **hcloud CLI** geïnstalleerd en geauthenticeerd (`hcloud context list` laat een actieve context zien)
- **Hetzner account** met een project aangemaakt in de Cloud Console
- **SSH keypair** op je lokale machine (bij voorkeur `~/.ssh/id_ed25519`)
- Toegang tot het terminal op je lokale machine

Controleer of de hcloud CLI werkt:

```bash
hcloud version
hcloud context list
```

---

## 1. SSH Key Registreren bij Hetzner

Hetzner maakt bij server-aanmaak een SSH key in de `authorized_keys` van root. Als je je key vooraf registreert, hoef je dit niet handmatig te doen na de eerste boot.

Controleer of je key al geregistreerd is:

```bash
hcloud ssh-key list
```

Als de key er niet tussen staat, voeg hem toe:

```bash
hcloud ssh-key create \
  --name "<sleutelnaam>" \
  --public-key "$(cat ~/.ssh/id_ed25519.pub)"
```

Kies een beschrijvende naam, bijvoorbeeld je machinehostnaam of gebruikersnaam.

---

## 2. VPS Aanmaken

Controleer eerst welke server-types beschikbaar zijn (Hetzner hernoemt typen periodiek):

```bash
hcloud server-type list
```

Kies een type met minimaal 4 vCPU en 8 GB RAM — momenteel `cx33` of equivalent. `nbg1` (Neurenberg) is een solide keuze voor Europese latency. Controleer de huidige naam in de lijst hierboven als `cx33` niet beschikbaar is.

```bash
hcloud server create \
  --name <SERVERNAAM> \
  --type cx33 \
  --image ubuntu-24.04 \
  --location nbg1 \
  --ssh-key <sleutelnaam> \
  --label service=<label>
```

Na aanmaak toont hcloud het publieke IP-adres. Sla dit op of haal het later op:

```bash
SSH_IP=$(hcloud server ip <SERVERNAAM>)
echo $SSH_IP
```

De `--label` vlag maakt het later makkelijk om servers te groeperen en te filteren (`hcloud server list -l service=<label>`).

---

## 3. Eerste Inlog en Systeem-update

Een verse Ubuntu-installatie bevat vrijwel altijd openstaande pakketupdates. Patch eerst alles voordat je verder gaat — zo verklein je het aanvalsoppervlak voor de stappen erna.

```bash
SSH_IP=$(hcloud server ip <SERVERNAAM>)
ssh root@$SSH_IP "apt-get update && apt-get upgrade -y"
```

Eventueel een reboot direct na de update als er een nieuwe kernel is geïnstalleerd:

```bash
ssh root@$SSH_IP "reboot"
```

Wacht 30–60 seconden en log daarna opnieuw in om te verifiëren:

```bash
ssh root@$SSH_IP "uname -r && uptime"
```

---

## 4. SSH Hardening

Wachtwoordauthenticatie is de grootste aanvalsvector op publieke servers. Bots scannen continu het internet op SSH-poort 22 en proberen zwakke wachtwoorden. Door alleen SSH-keys toe te staan — en het aantal inlogpogingen te beperken — sluit je deze aanvalsvector volledig af.

```bash
SSH_IP=$(hcloud server ip <SERVERNAAM>)
ssh root@$SSH_IP "cat > /etc/ssh/sshd_config.d/hardening.conf << 'SSHD'
PasswordAuthentication no
PermitRootLogin prohibit-password
MaxAuthTries 3
SSHD
systemctl restart sshd"
```

Wat deze instellingen doen:

| Instelling | Effect |
|---|---|
| `PasswordAuthentication no` | Alleen SSH-keys worden geaccepteerd |
| `PermitRootLogin prohibit-password` | Root mag inloggen, maar alleen met een key |
| `MaxAuthTries 3` | Na 3 mislukte pogingen wordt de verbinding verbroken |

Verifieer dat je nog steeds kunt inloggen na het herstarten van sshd:

```bash
ssh root@$SSH_IP "echo 'SSH hardening OK'"
```

---

## 5. Automatische Beveiligingsupdates

Handmatig patchen vergeet je. `unattended-upgrades` zorgt dat beveiligingsupdates automatisch worden geïnstalleerd. Kernelversies worden niet automatisch actief — dat vereist een handmatige reboot — maar pakketbeveiligingsupdates worden wel direct toegepast.

Controleer of de service al draait (Ubuntu 24.04 heeft dit meestal standaard actief):

```bash
SSH_IP=$(hcloud server ip <SERVERNAAM>)
ssh root@$SSH_IP "systemctl status unattended-upgrades"
```

Als de service niet actief is, installeer en activeer:

```bash
ssh root@$SSH_IP "apt-get install -y unattended-upgrades && systemctl enable --now unattended-upgrades"
```

---

## 6. Hostname Instellen

Een correcte hostname maakt logs en prompts leesbaar en vermijdt verwarring bij meerdere servers. Dit is optioneel maar netjes.

```bash
SSH_IP=$(hcloud server ip <SERVERNAAM>)
ssh root@$SSH_IP "hostnamectl set-hostname <SERVERNAAM>"
```

Verifieer:

```bash
ssh root@$SSH_IP "hostname"
```

---

## Vervolgstap

De VPS is nu aangemaakt en basisbeveiliging is op orde. Ga verder met:

**[02-Firewall.md](02-Firewall.md)** — UFW en Hetzner Cloud Firewall instellen.
