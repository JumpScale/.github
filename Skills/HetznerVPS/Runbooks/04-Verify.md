# 04-Verify  -  VPS Verificatie-Checklist

Dit runbook verifieert of de VPS-installatie volledig en correct is uitgevoerd.
Voer dit uit nadat alle gekozen runbooks zijn doorlopen.

---

## 1. Quick check

```bash
# Tailscale (als gekozen)
ssh root@<IP> "tailscale status --peers=false"
# Verwacht: Tailscale IP zichtbaar, status: running
```

---

## 2. Security checks

```bash
# Hetzner Cloud Firewall actief?
hcloud firewall list

# UFW actief?
ssh root@<IP> "ufw status | head -3"
# Verwacht: Status: active

# SSH key-only (geen wachtwoordlogin)?
ssh root@<IP> "sshd -T | grep passwordauthentication"
# Verwacht: passwordauthentication no

# Automatische updates actief?
ssh root@<IP> "systemctl is-active unattended-upgrades"
# Verwacht: active

# Geen onverwachte publieke poorten?
ssh root@<IP> "ss -tulnp | grep LISTEN | grep -v '127.0.0.1'"
# Verwacht: alleen poort 22 (en eventueel 41641 voor Tailscale)
```

---

## 3. Bereikbaarheidstest

```bash
# SSH werkt?
ssh root@<IP> "echo 'SSH OK'"

# Tailscale bereikbaar (als gekozen)?
ssh root@<IP> "tailscale ping $(tailscale ip -4) --timeout=5s"
```

---

## 4. Installatiechecklist

Vink af op basis van de gemaakte keuzes:

**Altijd:**
- [ ] VPS aangemaakt en bereikbaar via SSH
- [ ] Systeem volledig bijgewerkt (`apt-get upgrade`)
- [ ] SSH wachtwoordlogin uitgeschakeld
- [ ] MaxAuthTries ingesteld op 3
- [ ] Unattended-upgrades actief
- [ ] Hetzner Cloud Firewall actief (TCP 22, UDP 41641, ICMP)
- [ ] UFW actief (default deny, SSH limit)
- [ ] Hostname ingesteld

**Als Tailscale gekozen:**
- [ ] Tailscale geïnstalleerd en verbonden
- [ ] VPS zichtbaar in Tailscale console
- [ ] UFW regel voor tailscale0 toegevoegd
- [ ] UDP 41641 doorgelaten in Hetzner firewall

**Als GEEN Tailscale:**
- [ ] fail2ban geïnstalleerd en actief
- [ ] IP-restrictie overwogen
- [ ] HTTPS via Let's Encrypt klaar (indien services publiek)

---

## 5. Wat nu?

Na succesvolle verificatie:

1. **Noteer je gegevens veilig** (IP-adres, Tailscale-hostnaam)
2. **Installeer applicaties**  -  bijv. OpenClaw (zie de [OpenClaw skill](../../OpenClaw/))
3. **Periodiek onderhoud**  -  voer maandelijks de Maintain-workflow uit
