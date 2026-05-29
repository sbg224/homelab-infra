# homelab-infra

Infrastructure personnelle en production — réseau segmenté, virtualisation, services auto-hébergés.  
Tout tourne sur réseau physique local à Toulouse.

---

## Matériel

| Composant | Détail |
|---|---|
| Hyperviseur | Proxmox VE 8.4.19 |
| CPU | Intel Core i5-6500T @ 2.50GHz (4 cœurs) |
| RAM | 23.34 Go |
| Stockage système | 67.82 Go |
| Stockage NAS | Disque dur externe partitionné, monté sur Proxmox |
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

## Architecture

> *Schéma architecture*

![Architecture](docs/screenshots/architecture.png)

---

## Machines virtuelles

| VM | Nom | OS | Rôle |
|---|---|---|---|
| VM 100 | Router-opensens | OPNsense 25.7.3 | Routeur / Pare-feu |
| VM 101 | NAS | Debian 12 | NAS — OpenMediaVault + Nextcloud + Samba |
| VM 102 | Server-Debian | Debian 12 | VPN — WireGuard |
| VM 103 | pi-hole-DNS | Debian 12 | DNS — Pi-hole |
| VM 104 | VMIA | Debian 12 | IA & Automatisation |
| VM 200 | Docker-service | Debian 12 | Services Docker |

> *Interface Proxmox — VMs en production*

![Proxmox](docs/screenshots/proxmox.png)

---

## Réseau — OPNsense (VM 100)

### Bridges Proxmox

| Bridge | Rôle |
|---|---|
| vmbr0 | WAN — connexion réseau physique |
| vmbr1 | LAN — bridge interne avec tagging VLAN |

### Interfaces OPNsense

| Interface | Réseau | Usage |
|---|---|---|
| WAN | 192.168.1.254 (DHCP) | Connexion Internet |
| LAN | 192.168.2.x | Réseau local de gestion |
| vlan10 | 192.168.10.x | Réseau core (NAS, Proxmox) |
| vlan_docker | 192.168.20.x | Services Docker |
| vlan_test | 192.168.30.x | Tests & invités |
| VPN WireGuard | 10.10.0.1 | Accès distant |

- **DHCP** configuré par interface dans OPNsense
- **Routage inter-VLAN** avec règles de pare-feu par segment
- **NAT** sortant sur interface WAN

> *Dashboard OPNsense — interfaces et trafic*

![OPNsense](docs/screenshots/opnsense.png)

---

## VPN — WireGuard (VM 102)

- Tunnel WireGuard sur VM dédiée
- Réseau dédié : `10.10.0.0/24`
- Peers configurés avec clés publiques / privées
- Accès SSH depuis Mac via clé SSH à travers le tunnel

---

## NAS (VM 101)

- **OpenMediaVault** : gestion du stockage, partages réseau
- **Nextcloud** : cloud personnel auto-hébergé
- **Samba (SMB)** : partage de dossiers sur réseau local
  - Utilisé pour exposer le vault Obsidian à un agent IA local (accès mémoire étendu)
- Stockage : disque dur externe partitionné et alloué via Proxmox

---

## DNS — Pi-hole (VM 103)

- Serveur DNS local
- Filtrage publicitaire réseau : **517 140 domaines bloqués**
- **6 247 requêtes** traitées, 7% bloquées
- 3 clients actifs

> *Dashboard Pi-hole*

![Pi-hole](docs/screenshots/pihole.png)

---

## Services Docker (VM 200)

Stack Docker Compose — VLAN 20 (vlan_docker) :

| Conteneur | Rôle | Port |
|---|---|---|
| n8n | Workflows et automatisations | 5678 |
| nextcloud-redis | Cache Redis pour Nextcloud | — |
| npm | Nginx Proxy Manager — reverse proxy + SSL | 80/443 |
| portainer | Administration Docker | 9000/9443 |
| postgres_gm | Base de données PostgreSQL | 5432 |

> *Liste des conteneurs — Portainer*

![Portainer](docs/screenshots/portainer.png)

---

## VM IA & Automatisation (VM 104)

VM dédiée à la centralisation des scripts, agents et automatisations. *(En cours de construction)*

**Objectif :**
- Héberger tous les scripts Bash / Python
- Monitoring de l'infrastructure
- Déployer les agents IA → voir [`veille-tech`](https://github.com/sbg224/veille-tech)
- Isolé sur VLAN dédié

---

## Accès à distance

1. Connexion WireGuard VPN
2. SSH avec clé publique/privée depuis Mac
3. Accès aux interfaces web via tunnel VPN (Proxmox, OPNsense, Portainer, Nextcloud)

---

## Stack technique

![Proxmox](https://img.shields.io/badge/Proxmox-VE_8.4-E57000?style=flat&logo=proxmox&logoColor=white)
![Debian](https://img.shields.io/badge/Debian-12-A81D33?style=flat&logo=debian&logoColor=white)
![OPNsense](https://img.shields.io/badge/OPNsense-25.7-D94F00?style=flat&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=flat&logo=docker&logoColor=white)
![WireGuard](https://img.shields.io/badge/WireGuard-VPN-88171A?style=flat&logo=wireguard&logoColor=white)
![Pi-hole](https://img.shields.io/badge/Pi--hole-DNS-96060C?style=flat&logoColor=white)
![Nextcloud](https://img.shields.io/badge/Nextcloud-Cloud-0082C9?style=flat&logo=nextcloud&logoColor=white)
![n8n](https://img.shields.io/badge/n8n-Automation-EA4B71?style=flat&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-DB-4169E1?style=flat&logo=postgresql&logoColor=white)
![Bash](https://img.shields.io/badge/Bash-Scripting-4EAA25?style=flat&logo=gnubash&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.x-3776AB?style=flat&logo=python&logoColor=white)
![Cisco](https://img.shields.io/badge/Cisco-SG200--08-1BA0D7?style=flat&logo=cisco&logoColor=white)

---

## Ajouter les screenshots

Créer un dossier `docs/screenshots/` dans le repo et y déposer :

```
docs/
└── screenshots/
    ├── architecture.png   ← schéma Excalidraw
    ├── proxmox.png        ← dashboard Proxmox
    ├── opnsense.png       ← dashboard OPNsense
    ├── pihole.png         ← dashboard Pi-hole
    └── portainer.png      ← liste conteneurs Portainer
```

---

*Infrastructure personnelle en évolution continue*  
📍 Toulouse · [LinkedIn](https://www.linkedin.com/in/mohamed-bah-aa38a1232/) · [GitHub](https://github.com/sbg224)
