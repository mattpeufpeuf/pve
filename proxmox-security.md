# Sécurisation Proxmox VE

## 1. Mises à jour
```bash
apt update && apt upgrade -y
pveupgrade
```

## 2. Changer le mot de passe root + désactiver accès root direct si souhaité
- Changer dans Datacenter > Users ou `passwd`
- Créer utilisateur avec privilèges Admin, utiliser celui-ci

## 3. Interface et réseau
- Pas exposer 8006 sur WAN (firewall routeur)
- Accès uniquement LAN/VPN

## 4. Firewall Proxmox
Datacenter > Firewall:
- Activer pour Datacenter + noeud + VMs/CTs
- Règles entrantes strictes (SSH 22 LAN, 8006 LAN uniquement)
- Drop par défaut entrant

## 5. SSH
/etc/ssh/sshd_config.d/99-hardening.conf:
```
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
MaxAuthTries 3
LoginGraceTime 15
X11Forwarding no
AllowAgentForwarding no
PermitTunnel no
ClientAliveInterval 300
ClientAliveCountMax 2
```
`systemctl reload ssh`

## 6. Fail2ban
```bash
apt install fail2ban
cat > /etc/fail2ban/jail.d/proxmox.conf << 'J'
[pve]
enabled = true
port = 8006,22
filter = pve
logpath = /var/log/daemon.log
maxretry = 5
bantime = 1h
findtime = 10m
J
systemctl enable --now fail2ban
```

## 7. Mises à jour no-subscription (optionnel mais courant)
Remplacer /etc/apt/sources.list.d/pve-enterprise.list ou ajouter no-sub
Attention: uniquement si non sous licence entreprise.

## 8. 2FA
Datacenter > Permissions > Two Factor Authentication (TOTP/WebAuthn)

## 9. Journalisation / sauvegarde
- Activer sauvegardes Proxmox (PBS recommandé)
- Exporter config /etc/pve régulièrement

## 10. Contrôles rapides
```bash
pveversion -v
iptables -L -n
ss -tlnp
fail2ban-client status pve
```

## Notes
- Ne jamais exposer 8006 à Internet
- Privilégier VPN pour accès distant
- Surveiller logs /var/log/auth.log et syslog
