# homelab-infra

Documentation de mon infrastructure personnelle — réseau, virtualisation, services et automatisation.  
Tout tourne en production sur mon réseau physique local à Toulouse.

---

## Matériel

| Composant | Détail |
|---|---|
| Hyperviseur | Proxmox VE |
| CPU | 8 cœurs |
| RAM | 23 Go |
| Stockage | 256 Go (système) + disque externe (NAS) |
| Switch | Cisco SG200-08 |

**Réseau physique :**
```
Prise Ethernet (FAI)
        │
        ▼
[Switch Cisco SG200-08]
        │
   ┌────┴────┐
   ▼         ▼
[PC]    [Serveur Proxmox]
```

---

## Machines virtuelles

| VM | OS | Rôle |
|---|---|---|
| VM 100 | OPNsense | Routeur / Pare-feu |
| VM 101 | Debian 12 | NAS — Nextcloud + Samba |
| VM 102 | Debian 12 | VPN — WireGuard |
| VM 103 | Debian 12 | DNS — Pi-hole |
| VM 104 | Debian 12 | IA & Automatisation *(en cours)* |
| VM 200 | Debian 12 | Services Docker |

> Toutes les VMs tournent sous Debian 12.  
> Stockage NAS : disque dur externe monté sur Proxmox, partitionné et alloué aux services.

---

## Réseau — OPNsense (VM 100)

### Bridges Proxmox

| Bridge | Rôle |
|---|---|
| vmbr0 | WAN — connexion réseau physique |
| vmbr1 | LAN — bridge interne avec tagging VLAN |

Chaque VM reçoit un tag VLAN via vmbr1 pour être isolée sur le bon segment réseau.

### Interfaces OPNsense

| Interface | VLAN | Usage |
|---|---|---|
| LAN | — | Réseau local de gestion |
| VLAN 10 | 10 | WireGuard VPN |
| VLAN 20 | 20 | Docker & services |
| VLAN 30 | 30 | Invité / Test |

- **DHCP** configuré par interface dans OPNsense
- **Routage inter-VLAN** avec règles de pare-feu entre segments
- **NAT** sortant sur interface WAN

---

## VPN — WireGuard (VM 102)

- Tunnel WireGuard sur VM dédiée (VLAN 10)
- Peers configurés avec clés publiques / privées
- Permet l'administration à distance de toute l'infrastructure
- Accès SSH depuis Mac via clé SSH à travers le tunnel

---

## NAS — Nextcloud + Samba (VM 101)

- **Nextcloud** : stockage fichiers personnel, cloud auto-hébergé
- **Samba (SMB)** : partage de dossiers sur le réseau local
  - Utilisé notamment pour exposer le vault Obsidian à un agent IA local (OpenClaw), lui fournissant un contexte mémoire étendu
- Stockage sur disque externe partitionné et monté via Proxmox

---

## DNS — Pi-hole (VM 103)

- Serveur DNS local sur VLAN dédié
- Filtrage publicitaire au niveau réseau
- Résolution DNS interne pour les services auto-hébergés

---

## Services Docker (VM 200)

Stack Docker Compose sur VLAN 20 :

| Service | Rôle |
|---|---|
| n8n | Workflows et automatisations |
| Nginx Proxy Manager | Reverse proxy + certificats SSL |
| Portainer | Administration de la stack Docker |
| PostgreSQL | Base de données relationnelle |

```bash
# Vérifier l'état des services
docker compose ps

# Redémarrer un service
docker compose restart <service>
```

---

## VM IA & Automatisation (VM 104)

VM dédiée à la centralisation des scripts, agents et automatisations. *(En cours de construction)*

**Objectif :**
- Héberger tous les nouveaux scripts Bash / Python
- Centraliser le monitoring de l'infrastructure
- Déployer les agents IA (dont VeilleBot → voir [`veille-tech`](https://github.com/sbg224/veille-tech))
- Isoler les automatisations du reste des services (VLAN 30)

---

## Accès à distance

1. Connexion WireGuard VPN (depuis n'importe où)
2. SSH avec clé publique/privée vers les VMs cibles
3. Accès aux interfaces web (Proxmox, OPNsense, Portainer, Nextcloud) via tunnel VPN

---

## Stack technique

![Proxmox](https://img.shields.io/badge/Proxmox-VE-E57000?style=flat&logo=proxmox)
![Debian](https://img.shields.io/badge/Debian-12-A81D33?style=flat&logo=debian)
![OPNsense](https://img.shields.io/badge/OPNsense-Firewall-D94F00?style=flat)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=flat&logo=docker)
![WireGuard](https://img.shields.io/badge/WireGuard-VPN-88171A?style=flat&logo=wireguard)
![Pi--hole](https://img.shields.io/badge/Pi--hole-DNS-96060C?style=flat)
![Nextcloud](https://img.shields.io/badge/Nextcloud-NAS-0082C9?style=flat&logo=nextcloud)
![Bash](https://img.shields.io/badge/Bash-Scripting-4EAA25?style=flat&logo=gnubash)
![Python](https://img.shields.io/badge/Python-3.x-3776AB?style=flat&logo=python)
![Cisco](https://img.shields.io/badge/Cisco-SG200--08-1BA0D7?style=flat&logo=cisco)

---

*Infrastructure personnelle en évolution continue*  
📍 Toulouse · [LinkedIn](https://www.linkedin.com/in/mohamed-bah-aa38a1232/) · [GitHub](https://github.com/sbg224)
