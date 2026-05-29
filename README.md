# homelab-infra

Documentation de mon infrastructure personnelle — réseau, virtualisation, services et automatisation.  
Tout est en production sur mon réseau physique local.

---

## Vue d'ensemble

| Composant | Technologie | Rôle |
|---|---|---|
| Hyperviseur | Proxmox VE | Hébergement des VMs |
| Pare-feu | OPNsense | Routage, firewall, NAT, VLANs |
| VPN | WireGuard | Accès distant sécurisé |
| DNS | Pi-hole | Filtrage DNS local |
| Conteneurs | Docker Compose | Services auto-hébergés |
| Switch | Cisco SG200-08 | Commutation réseau |

---

## Architecture réseau

```
Internet
    │
    ▼
[Modem/FAI]
    │
    ▼
[OPNsense — Pare-feu / Routeur]
    │
    ├── VLAN 10 — WireGuard VPN
    │       └── VM WireGuard (accès distant)
    │
    ├── VLAN 20 — Docker & services
    │       └── VM Docker (Nextcloud, Nginx, Portainer, n8n, Pi-hole)
    │
    └── VLAN 30 — Automatisation & scripts
            └── VM Python/n8n (scripts, veille, agents)

[Switch Cisco SG200-08] — ports taggés / non-taggés par VLAN
```

---

## Machines virtuelles

| VM | OS | IP | Rôle |
|---|---|---|---|
| opnsense | OPNsense | 192.168.1.1 | Pare-feu, routeur, DHCP |
| wireguard | Debian 12 | 192.168.10.x | Serveur VPN WireGuard |
| docker-host | Debian 12 | 192.168.20.10 | Stack Docker |
| automation | Debian 12 | 192.168.30.x | Scripts Python, n8n |
| pihole | Debian 12 | 192.168.20.x | DNS Pi-hole |

> Hyperviseur : Proxmox VE — accès via API REST et interface web

---

## Réseau — OPNsense

### Segmentation VLAN

| VLAN | Réseau | Usage |
|---|---|---|
| VLAN 10 | 192.168.10.0/24 | WireGuard VPN |
| VLAN 20 | 192.168.20.0/24 | Docker & services |
| VLAN 30 | 192.168.30.0/24 | Automatisation & scripts |

### Règles firewall
- Isolation entre VLANs (pas de communication inter-VLAN non autorisée)
- NAT sortant sur l'interface WAN
- Accès VLAN 10 → VLAN 20 autorisé (administration via VPN)

---

## VPN — WireGuard

- Hébergé sur VM dédiée (VLAN 10)
- Intégré au pare-feu OPNsense
- Permet l'administration à distance de l'infrastructure
- Gestion des peers et rotation des clés

---

## Stack Docker

Services déployés sur la VM `docker-host` (VLAN 20) :

| Service | Image | Usage |
|---|---|---|
| Nextcloud | `nextcloud` | Stockage fichiers |
| Nginx Proxy Manager | `jc21/nginx-proxy-manager` | Reverse proxy + SSL |
| Portainer | `portainer/portainer-ce` | Administration Docker |
| n8n | `n8nio/n8n` | Workflows automatisés |
| Pi-hole | `pihole/pihole` | DNS + filtrage publicités |

```bash
# Démarrage de la stack
docker compose up -d

# Vérification des conteneurs
docker compose ps
```

---

## Scripts d'automatisation

Tous les scripts sont en Bash et Python, déployés via `cron` ou `systemd`.

### Mises à jour automatiques
```bash
# /scripts/update.sh
#!/bin/bash
apt update && apt upgrade -y
docker compose pull && docker compose up -d
```

### Surveillance des conteneurs
```bash
# /scripts/monitor.sh — vérifie que les conteneurs sont UP
# Envoie une alerte Telegram si un service est down
```

### Sauvegardes volumes Docker
```bash
# /scripts/backup.sh — archive les volumes Docker vers stockage local
```

### Snapshots Proxmox
```bash
# Snapshot automatique des VMs via l'API REST Proxmox
# Planifié via cron toutes les nuits
```

---

## Administration

### Proxmox
- Accès : `https://192.168.1.x:8006`
- API REST utilisée pour snapshots et supervision
- Gestion des ressources CPU/RAM par VM

### Accès à distance
1. Connexion WireGuard VPN
2. SSH vers la VM cible
3. Ou accès aux interfaces web via tunnel VPN

---

## Stack technique

![Linux](https://img.shields.io/badge/Linux-Debian_12-informational?style=flat&logo=debian&color=A81D33)
![Proxmox](https://img.shields.io/badge/Proxmox-VE-informational?style=flat&logo=proxmox&color=E57000)
![Docker](https://img.shields.io/badge/Docker-Compose-informational?style=flat&logo=docker&color=2496ED)
![OPNsense](https://img.shields.io/badge/OPNsense-Firewall-informational?style=flat&color=D94F00)
![WireGuard](https://img.shields.io/badge/WireGuard-VPN-informational?style=flat&logo=wireguard&color=88171A)
![Bash](https://img.shields.io/badge/Bash-Scripting-informational?style=flat&logo=gnubash&color=4EAA25)
![Python](https://img.shields.io/badge/Python-3.x-informational?style=flat&logo=python&color=3776AB)

---

*Infrastructure personnelle — en évolution continue*  
📍 Toulouse · [LinkedIn](https://www.linkedin.com/in/mohamed-bah-aa38a1232/) · [GitHub](https://github.com/sbg224)
