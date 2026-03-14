# Maintain Workflow

Periodiek onderhoud voor een bestaande Hetzner VPS.
Aanbevolen: eens per maand uitvoeren.

---

## Fase 0: Verbinding opzetten

Vraag de gebruiker naar de servernaam of het IP-adres van de VPS:

```bash
# Ophalen via hcloud als je de servernaam weet:
IP=$(hcloud server ip <SERVERNAAM>)
echo "VPS IP: $IP"
```

---

## Fase 1: Research

Voer eerst de research-stap uit. Lees en volg: `../Research.md`

---

## Fase 2: Status-check

Voer alle checks uit en presenteer de uitkomsten aan de gebruiker.

### Firewall

```bash
# Hetzner Cloud Firewall
hcloud firewall list

# UFW op de VPS
ssh root@<IP> "ufw status verbose"
```

### SSH-configuratie

```bash
ssh root@<IP> "sshd -T | grep -E 'passwordauthentication|permitrootlogin|maxauthtries'"
```

### Automatische updates

```bash
ssh root@<IP> "systemctl is-active unattended-upgrades && stat -c '%y' /var/lib/apt/periodic/update-success-stamp 2>/dev/null"
```

### Beschikbare updates

```bash
ssh root@<IP> "apt list --upgradable 2>/dev/null | grep -c -v 'Listing'"
```

### Disk en geheugen

```bash
ssh root@<IP> "df -h / && free -h"
```

### Reboot nodig?

```bash
ssh root@<IP> "test -f /var/run/reboot-required && echo 'REBOOT AANBEVOLEN' || echo 'Geen reboot nodig'"
```

### Tailscale (als geïnstalleerd)

```bash
ssh root@<IP> "tailscale status --peers=false 2>/dev/null || echo 'Tailscale niet geïnstalleerd'"
```

### fail2ban (als geïnstalleerd)

```bash
ssh root@<IP> "fail2ban-client status 2>/dev/null || echo 'fail2ban niet geïnstalleerd'"
```

---

## Fase 3: Acties

Op basis van de status-check:

### Updates installeren (als beschikbaar)

```bash
ssh root@<IP> "apt-get update && apt-get upgrade -y"
```

### Reboot uitvoeren (als aanbevolen)

```bash
ssh root@<IP> "reboot"
# Wacht 30-60 seconden, dan opnieuw inloggen
```

### Docker images opruimen

```bash
ssh root@<IP> "docker image prune -f 2>/dev/null || echo 'Docker niet geïnstalleerd'"
```

---

## Maandelijkse checklist

- [ ] Firewall-regels intact (Hetzner + UFW)
- [ ] SSH key-only actief
- [ ] Automatische updates actief
- [ ] Handmatige updates geïnstalleerd
- [ ] Disk niet vol (< 80% gebruik)
- [ ] Reboot uitgevoerd indien aanbevolen
- [ ] Docker images opgeruimd (als Docker aanwezig)
