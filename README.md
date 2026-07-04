Compris, je te donne le fichier entier en une seule fois, du tout premier caractère au dernier, pour que tu remplaces la totalité du contenu de ton `README.md` d'un coup et qu'on évite le problème de collage partiel de tout à l'heure.

```markdown
# homelab-infra

Infrastructure personnelle en production, réseau segmenté, virtualisation, services auto-hébergés.
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

**Réseau physique**

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
| VM 101 | NAS | Debian 12 | NAS avec OpenMediaVault, Nextcloud et Samba |
| VM 102 | Server-Debian | Debian 12 | VPN WireGuard |
| VM 103 | pi-hole-DNS | Debian 12 | DNS avec Pi-hole |
| VM 105 | WS | Windows Server 2022 | Contrôleur de domaine Active Directory |
| VM 106 | WIN10-CLIENT | Windows 10 | Poste client joint au domaine |
| VM 200 | Docker-service | Debian 12 | Services Docker |

> *Interface Proxmox VMs en production*

![Proxmox](docs/screenshots/proxmox.png)

---

## Réseau OPNsense (VM 100)

### Bridges Proxmox

| Bridge | Rôle |
|---|---|
| vmbr0 | WAN connexion réseau physique |
| vmbr1 | LAN bridge interne avec tagging VLAN |

### Interfaces OPNsense

| Interface | Réseau | Usage |
|---|---|---|
| WAN | 192.168.1.254 (DHCP) | Connexion Internet |
| LAN | 192.168.2.x | Réseau local de gestion |
| vlan10 | 192.168.10.x | Réseau core (NAS, Proxmox) |
| vlan_docker | 192.168.20.x | Services Docker |
| vlan_test | 192.168.30.x | Tests et invités |
| VPN WireGuard | 10.10.0.1 | Accès distant |

- **DHCP** configuré par interface dans OPNsense
- **Routage inter-VLAN** avec règles de pare-feu par segment
- **NAT** sortant sur interface WAN

> *Dashboard OPNsense interfaces et trafic*

![OPNsense](docs/screenshots/opnsense.png)

---

## VPN WireGuard (VM 102)

- Tunnel WireGuard sur VM dédiée
- Réseau dédié `10.10.0.0/24`
- Peers configurés avec clés publiques / privées
- Accès SSH depuis Mac via clé SSH à travers le tunnel

---

## Lab Windows Server et Active Directory (VM 105 et VM 106)

- Installation de Windows Server 2022 sur Proxmox (VM 105, nommée WS)
- Promotion du serveur en contrôleur de domaine, nom de domaine msblab.local
- Création d'unités d'organisation et d'utilisateurs dans Active Directory
- Mise en place de stratégies de groupe (GPO)
- Jonction du poste client Windows 10 (VM 106, nommée WIN10-CLIENT) au domaine

> *Console Active Directory et jonction du poste client*

*[capture à ajouter]*

---

## NAS (VM 101)

- **OpenMediaVault** pour la gestion du stockage et des partages réseau
- **Nextcloud**, cloud personnel auto-hébergé
- **Samba (SMB)** pour le partage de dossiers sur le réseau local
  - Utilisé pour exposer le vault Obsidian à un agent IA local (accès mémoire étendu)
- Stockage sur disque dur externe partitionné et alloué via Proxmox

---

## DNS Pi-hole (VM 103)

- Serveur DNS local
- Filtrage publicitaire réseau, **517 140 domaines bloqués**
- **6 247 requêtes** traitées, 7% bloquées
- 3 clients actifs

> *Dashboard Pi-hole*

![Pi-hole](docs/screenshots/pihole.png)

---

## Services Docker (VM 200)

Stack Docker Compose sur le VLAN 20 (vlan_docker)

| Conteneur | Rôle | Port |
|---|---|---|
| n8n | Workflows et automatisations | 5678 |
| nextcloud-redis | Cache Redis pour Nextcloud | aucun |
| npm | Nginx Proxy Manager, reverse proxy et SSL | 80/443 |
| portainer | Administration Docker | 9000/9443 |
| postgres_gm | Base de données PostgreSQL | 5432 |

> *Liste des conteneurs Portainer*

![Portainer](docs/screenshots/portainer.png)

---

## Veille technique et automatisation (service)

Service de centralisation des scripts, agents et automatisations, hébergé désormais au sein d'une des VM existantes de l'infrastructure plutôt que sur une VM dédiée. *(En cours de construction)*

**Objectif**
- Héberger tous les scripts Bash / Python
- Monitoring de l'infrastructure
- Déployer les agents IA, voir [`veille-tech`](https://github.com/sbg224/veille-tech)
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
![Windows Server](https://img.shields.io/badge/Windows_Server-2022-0078D6?style=flat&logo=windows&logoColor=white)
![Active Directory](https://img.shields.io/badge/Active_Directory-Domaine-0078D6?style=flat&logoColor=white)

---

*Infrastructure personnelle en évolution continue*
📍 Toulouse · [LinkedIn](https://www.linkedin.com/in/mohamed-bah-aa38a1232/) · [GitHub](https://github.com/sbg224)
```

Deux choses à vérifier une fois collé.

J'ai retiré la ligne d'image `windows-server-ad.png` dans la section du lab Windows Server puisque tu n'avais pas confirmé que ce fichier existe, et je l'ai remplacée par un simple texte entre crochets en attendant une vraie capture. Si tu en prends une plus tard, dis-le-moi et on la réintègre proprement.

Sur GitHub, ouvre l'éditeur du fichier, sélectionne tout le contenu existant avec Ctrl A ou Cmd A, supprime, puis colle ce bloc en entier. Vérifie que la toute première ligne visible est bien `# homelab-infra`, sans rien avant. Actualise ensuite la prévisualisation pour confirmer que l'erreur rouge a disparu.

Dis-moi quand c'est fait, et on coche la case 1.1.---

## Architecture

> *Schéma architecture*

![Architecture](docs/screenshots/architecture.png)

---

## Machines virtuelles

| VM | Nom | OS | Rôle |
|---|---|---|---|
| VM 100 | Router-opensens | OPNsense 25.7.3 | Routeur / Pare-feu |
| VM 101 | NAS | Debian 12 | NAS avec OpenMediaVault, Nextcloud et Samba |
| VM 102 | Server-Debian | Debian 12 | VPN WireGuard |
| VM 103 | pi-hole-DNS | Debian 12 | DNS avec Pi-hole |
| VM 105 | WS | Windows Server 2022 | Contrôleur de domaine Active Directory |
| VM 106 | WIN10-CLIENT | Windows 10 | Poste client joint au domaine |
| VM 200 | Docker-service | Debian 12 | Services Docker |

> *Interface Proxmox VMs en production*

![Proxmox](docs/screenshots/proxmox.png)

---

## Réseau OPNsense (VM 100)

### Bridges Proxmox

| Bridge | Rôle |
|---|---|
| vmbr0 | WAN connexion réseau physique |
| vmbr1 | LAN bridge interne avec tagging VLAN |

### Interfaces OPNsense

| Interface | Réseau | Usage |
|---|---|---|
| WAN | 192.168.1.254 (DHCP) | Connexion Internet |
| LAN | 192.168.2.x | Réseau local de gestion |
| vlan10 | 192.168.10.x | Réseau core (NAS, Proxmox) |
| vlan_docker | 192.168.20.x | Services Docker |
| vlan_test | 192.168.30.x | Tests et invités |
| VPN WireGuard | 10.10.0.1 | Accès distant |

- **DHCP** configuré par interface dans OPNsense
- **Routage inter-VLAN** avec règles de pare-feu par segment
- **NAT** sortant sur interface WAN

> *Dashboard OPNsense interfaces et trafic*

![OPNsense](docs/screenshots/opnsense.png)

---

## VPN WireGuard (VM 102)

- Tunnel WireGuard sur VM dédiée
- Réseau dédié `10.10.0.0/24`
- Peers configurés avec clés publiques / privées
- Accès SSH depuis Mac via clé SSH à travers le tunnel

---

## Lab Windows Server et Active Directory (VM 105 et VM 106)

- Installation de Windows Server 2022 sur Proxmox (VM 105, nommée WS)
- Promotion du serveur en contrôleur de domaine, nom de domaine msblab.local
- Création d'unités d'organisation et d'utilisateurs dans Active Directory
- Mise en place de stratégies de groupe (GPO)
- Jonction du poste client Windows 10 (VM 106, nommée WIN10-CLIENT) au domaine

> *Console Active Directory et jonction du poste client*

![Lab Windows Server](docs/screenshots/windows-server-ad.png)

*[capture à ajouter si le fichier ci-dessus n'existe pas encore]*

---

## NAS (VM 101)

- **OpenMediaVault** pour la gestion du stockage et des partages réseau
- **Nextcloud**, cloud personnel auto-hébergé
- **Samba (SMB)** pour le partage de dossiers sur le réseau local
  - Utilisé pour exposer le vault Obsidian à un agent IA local (accès mémoire étendu)
- Stockage sur disque dur externe partitionné et alloué via Proxmox

---

## DNS Pi-hole (VM 103)

- Serveur DNS local
- Filtrage publicitaire réseau, **517 140 domaines bloqués**
- **6 247 requêtes** traitées, 7% bloquées
- 3 clients actifs

> *Dashboard Pi-hole*

![Pi-hole](docs/screenshots/pihole.png)

---

## Services Docker (VM 200)

Stack Docker Compose sur le VLAN 20 (vlan_docker)

| Conteneur | Rôle | Port |
|---|---|---|
| n8n | Workflows et automatisations | 5678 |
| nextcloud-redis | Cache Redis pour Nextcloud | aucun |
| npm | Nginx Proxy Manager, reverse proxy et SSL | 80/443 |
| portainer | Administration Docker | 9000/9443 |
| postgres_gm | Base de données PostgreSQL | 5432 |

> *Liste des conteneurs Portainer*

![Portainer](docs/screenshots/portainer.png)

---

## Veille technique et automatisation (service)

Service de centralisation des scripts, agents et automatisations, hébergé désormais au sein d'une des VM existantes de l'infrastructure plutôt que sur une VM dédiée. *(En cours de construction)*

**Objectif**
- Héberger tous les scripts Bash / Python
- Monitoring de l'infrastructure
- Déployer les agents IA, voir [`veille-tech`](https://github.com/sbg224/veille-tech)
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
![Windows Server](https://img.shields.io/badge/Windows_Server-2022-0078D6?style=flat&logo=windows&logoColor=white)
![Active Directory](https://img.shields.io/badge/Active_Directory-Domaine-0078D6?style=flat&logoColor=white)

---

*Infrastructure personnelle en évolution continue*
📍 Toulouse · [LinkedIn](https://www.linkedin.com/in/mohamed-bah-aa38a1232/) · [GitHub](https://github.com/sbg224)
