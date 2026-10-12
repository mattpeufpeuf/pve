# VPN + SSH + LAN notes

## Accès distant
- Accès via VPN uniquement (WireGuard/OpenVPN)
- Ne pas ouvrir 22/8006 sur WAN
- Depuis VPN, autoriser accès à 192.168.1.220:22 et 192.168.1.51/53/...:8006 selon besoin

## SSH hardening
Fichier: /etc/ssh/sshd_config.d/99-hardening.conf
- PermitRootLogin no
- PasswordAuthentication no (après clés)
- PubkeyAuthentication yes
- MaxAuthTries 3
- LoginGraceTime 15

## UFW (ce serveur)
Règle effective: allow 22/tcp depuis 192.168.1.0/24 uniquement
Vérif: `ufw status verbose`

## Routage VPN
ProtonVPN utilise policy routing → LAN intact (SSH 192.168.1.9 -> 192.168.1.220 OK)

## Changement IP VPN
Commandes:
```bash
XDG_RUNTIME_DIR=/run/user/0 dbus-run-session -- protonvpn connect --random
XDG_RUNTIME_DIR=/run/user/0 dbus-run-session -- protonvpn connect --fastest
```

## Vérifs
```bash
curl -s https://api.ipify.org
ip route | grep -v cache
ss -tnp '( sport = :22 )'
nmcli con show --active
```
