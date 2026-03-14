# 01-Docker — Docker installeren

Docker is de containerruntime waarin OpenClaw draait. Dit runbook controleert of Docker
aanwezig is en installeert het indien nodig.

---

## 1. Controleer of Docker al aanwezig is

```bash
ssh root@<IP> "docker --version && docker compose version"
```

- **Aanwezig:** Ga door naar [02-OpenClaw.md](02-OpenClaw.md).
- **Niet aanwezig:** Ga door naar stap 2.

---

## 2. Docker installeren

```bash
ssh root@<IP> << 'EOF'
apt-get update
apt-get install -y docker.io docker-compose-plugin
systemctl enable --now docker
docker --version
docker compose version
EOF
```

---

## 3. Docker log rotation instellen

Voorkom dat Docker-logs de disk volschrijven:

```bash
ssh root@<IP> "cat > /etc/docker/daemon.json" << 'EOF'
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
EOF
ssh root@<IP> "systemctl restart docker"
```

---

## Vervolgstap

Ga verder met: **[02-OpenClaw.md](02-OpenClaw.md)** — OpenClaw installeren en configureren.
