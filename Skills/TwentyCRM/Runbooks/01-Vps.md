# Stap 1: Server voorbereiden

## Doel
Een schone server met Docker, firewall en basisbeveiliging.

## Vereisten
- SSH root-toegang tot de server
- Ubuntu 22.04+ of Debian 12+

---

## 1.1 Systeem updaten [Claude doet dit]

```bash
apt update && apt upgrade -y
```

## 1.2 Docker installeren [Claude doet dit]

Controleer of Docker al geinstalleerd is:
```bash
docker --version && docker compose version
```

Zo niet:
```bash
curl -fsSL https://get.docker.com | sh
```

## 1.3 Firewall instellen [Claude doet dit]

```bash
ufw default deny incoming
ufw default allow outgoing
ufw allow 22/tcp
ufw allow 80/tcp
ufw allow 443/tcp
ufw --force enable
```

## 1.4 Fail2ban installeren [Claude doet dit]

```bash
apt install -y fail2ban
systemctl enable fail2ban
systemctl start fail2ban
```

## 1.5 Werkdirectories aanmaken [Claude doet dit]

```bash
mkdir -p /opt/twenty-crm /opt/n8n /opt/backups
```

---

## Verificatie

```bash
docker --version         # Docker geinstalleerd
ufw status              # Firewall actief, poorten 22/80/443 open
systemctl is-active fail2ban  # fail2ban actief
ls /opt/twenty-crm      # Directory bestaat
```
