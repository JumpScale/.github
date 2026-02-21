# 01 -- Server voorbereiden

## Doel

Server klaar voor Trigger.dev installatie: OS up-to-date, Docker geinstalleerd, firewall actief.
Trigger.dev vereist minimaal 8 GB RAM vanwege de meerdere services die tegelijk draaien.

---

## Stap 1: Systeem updaten

```bash
ssh root@<SERVER_IP> << 'EOF'
apt update && apt upgrade -y
apt install -y curl git ufw
EOF
```

## Stap 2: Docker installeren

```bash
ssh root@<SERVER_IP> << 'EOF'
# Docker installeren (als nog niet aanwezig)
if ! command -v docker &>/dev/null; then
    curl -fsSL https://get.docker.com | sh
fi

# Docker Compose plugin controleren
docker compose version

# Docker service starten
systemctl enable docker
systemctl start docker
EOF
```

## Stap 3: Firewall configureren

```bash
ssh root@<SERVER_IP> << 'EOF'
ufw default deny incoming
ufw default allow outgoing
ufw allow 22/tcp    # SSH
ufw allow 80/tcp    # HTTP (redirect naar HTTPS)
ufw allow 443/tcp   # HTTPS
ufw --force enable
ufw status verbose
EOF
```

## Stap 4: Werkmap aanmaken

```bash
ssh root@<SERVER_IP> "mkdir -p /opt/trigger-dev"
```

## Verificatie

```bash
ssh root@<SERVER_IP> << 'EOF'
echo "=== OS ==="
cat /etc/os-release | head -2
echo "=== Docker ==="
docker --version
docker compose version
echo "=== RAM ==="
free -h
echo "=== Firewall ==="
ufw status
echo "=== Werkmap ==="
ls -la /opt/trigger-dev/
EOF
```

Verwacht:
- OS up-to-date
- Docker + Compose geinstalleerd
- Minimaal 8 GB RAM beschikbaar
- UFW actief met poorten 22, 80, 443 open
- `/opt/trigger-dev/` bestaat
