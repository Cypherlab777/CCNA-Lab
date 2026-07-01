# Lab 10 : Network security hardening

## Overview

Ce lab est est oriente envers la securite d'une petite entreprise utilisant des technologies Cisco
Le but, dans un premier temps et de rendre fonctionnel le reseau et dans un deuxieme temps d'appliquer les politiques de securite
Cette topologies inclu un vlan utilisateur, un vlan admin/management, des vlans de departement et aussi un vlan de server hebergeant les services pour l'entreprise 

Technologies implementees :

- VLAN Segmentation
- Inter-VLAN Routing
- BPDU Guard / PortFast
- Rapid PVST+
- OSPF
- DHCP Relay
- DNS / HTTP Services
- TFTP Backup
- Extended ACLs
- VTY Access Control
- SSH Management
- RADIUS / AAA
- Syslog
- Port Security
- DHCP Snooping
- Dynamic ARP Inspection

---

## Objectives

- Construire un reseau multi-vlan complet utilisant des services centralises 
- Utiliser les ACLs pour appliquer les politiques de restrictions inter-vlans
- Restrindre la gestions des equipements au seul vlan 20
- Implementer ssh via RADIUS mais aussi via login local
- Centraliser les logs dans un serveur Syslog
- Sauvegarder les configurations dans equipements dans un serveur TFTP
- Proteger les switchs avec port-security et sticky mac-address ainsi que le switch serveur via SecureConfigured
- Prevenir les attaques rogue DHCP avec DHCP snooping
- Proteger les switchs contres les attaques arp spoofing et poisoning via DAI
- Valider le bon fonctionnement de la configuration et des securite misent en place 

---

## Topology Overview

La topologie est constitue de plusieurs vlan connecter via OSPF

Le vlan server quand a lui est constitue exclusivement des services suivant :

- DHCP Server
- DNS Server
- HTTP Server
- Syslog Server
- TFTP Server
- RADIUS Server

Le vlan 20 Admin a un access totale a tous les vlans ainsi qu'a tous les equipements pour des besoins de maintenance et de check up

![Topology](screenshots/Topology/Topology.png)

---

## Network design and Addressing

### Vlan Design 

| Vlan | Name | Network | Role |
|---|---|---|---|
| 10 | Users | 192.168.10.0/24  | Standard Users |
| 20 | Admin | 192.168.20.0/24 | Network Administration |
| 30 | RH | 192.168.30.0/24 | Humain Ressources department |
| 40 | DR | 192.168.40.0/24 | Direction department |
| 50 | Server | 192.168.50.0/24 | Infrastructure Services |
| 999 | Blackhole | N/A | Unused ports |

### Server Vlan Services

| Service | IP Address | Purpose |
|---|---|---|
| SYSLOG | 192.168.50.100 | Centralized loggin service |
| DNS | 192.168.50.101 | Centralized domain name Service |
| DHCP | 192.168.50.102 | Centralized DHCP service |
| TFTP | 192.168.50.103 | Backup Configuration service |
| HTTP | 192.168.50.104 | Web Service |
| RADIUS | 192.168.50.105 | Centralized authentification service |

---

## Security Policy

### Inter-Vlan Access Policy

| Source VLAN | Allowed Access | Restricted Access |
|---|---|---|
| VLan 10 | DNS - DHCP - TFTP - HTTP | Vlan 20 - Vlan 30 - Vlan 40 - Syslog - SSH Devices |
| VLan 20 | Full Access | None |
| VLan 30 | DNS - DHCP - TFTP - HTTP | Vlan 20 - Vlan 40 - Syslog - SSH Devices |
| VLan 40 | DNS - DHCP - TFTP - HTTP | Vlan 20 - Vlan 30 - Syslog - SSH Devices |
| VLan 50 | Services provider | No unrestricted user access |

### Management Access Policy 

L'access au equipement est exclusivement reserver au vlan 20 Admin via SSH

L'authentification se fait via le serveur RADIUS a l'exeption du R2, en cas de down du Server radius, le login local reste disponible 

### ACLs Design note

Pour les besoins du lab j'ai configurer les ACLs comme suit : 

1. Ce qui est permit l'est explicitement 
2. Ce qui est restreint l'est explicitement egalement 
3. le permit final autorise le reste des communications

---

## Security 

### Extended ACLs

Les ACLs etendue sont appliquer au plus pres de la source, ce type d'ACL etant beaucoup plus precise, cela ne risque pas d'autoriser ou de refuser du traffic non desire 
Les ACLs sont principalement pour des restrictions inter-vlan mais aussi pour les sercices

![R3 Show Access-lists](screenshots/verification/Acls/R3-Show-Access-Lists.png)

### SSH / RADIUS

SSH est utilise pour le management a distance
Un serveur RADIUS authentifie egalement les connections aux equipements, en cas de down le login local reste disponible 

### VTY Acces Control

Une ACLs a ete appliquer sur chaque VTY de chaque equipement pour n'autoriser que le Vlan 20 a s'y connecter.

### Syslog

Un Serveur Syslog a ete mis en place afin de recuperer et de centraliser tous les evenements important.
A noter que sur packet tracer il n'y a que l'option de log debugging qui etait disponible

### TFTP Backup

Un Serveur TFTP a ete mis en place afin d'y sauvegarder les configurations des equipements, cela facilite la gestion mais aussi offre plus de securite en cas de probleme 

### Port security

La fonctionnalite port-security est activer sur les ports access utilisateurs afin d'eviter tout rogue device non autorise

### DHCP Snooping

DHCP snooping a ete appliquer sur tous les switchs afin de prevenir un rogue DHCP

Seule les ports de confiances sont passe en trusted commme les uplink vers les routeurs

La limitation des paquets par secondes a 10 a ete appliquer sur TOUS les ports des Switchs afin d'eviter qu'un port utilisateur n'envoyent un trop grand nombre de requete DHCP

### Dynamic Arp Inspection

DAI a egalement ete activer sur TOUS les ports des switchs afin d'eviter les attaques arp spoofing et poisoning 

### Remark

Dynamic Arp inspection se basant sur la DHCP snooping binding table, j'ai d'abord active DHCP snooping sur les switchs et j'ai attendu que les tables se remplicent pour eviter que DAI ne refuse du trafic legitime en bloquant des macs non desire

---

## Full Validation

| Feature | Test | Expected Result | Actual Result | Evidence |
|---|---|---|---|---|
| RADIUS | Admin vlan connects to SW10 via SSH | Connetions | Success | [RADIUS Server](screenshots/verification/SSH-RADIUS/RADIUS-Server.png) [SSH Test](screenshots/verification/SSH-RADIUS/PC3-SSH-SW10.png)  |
| SYSLOG | Log received on server | Events log | Success | [Syslog Server](screenshots/verification/Syslog/Syslog.png) |
| TFTP | Configurations backed up | Files saved | Success | [Tftp Server](screenshots/verification/Tftp/Tftp.png) |
| HTTP | Web server reachable using DNS | HTTP server working | Success | [WEB Access](screenshots/verification/HTTP/PC5-HTTP.png) |
| DHCP | Clients received IPs | DHCP assignment works | Success | [DHCP Server](screenshots/verification/DHCP/DHCP.png) [DHCP assignement](screenshots/verification/DHCP/PC8-DHCP-Assignement.png) |
| DNS | Internal records resolve correctly | Internal records working | Success | [DNS Server](screenshots/verification/DNS/DNS.png) [Ping records successful](screenshots/verification/DNS/PC3-Ping-Dns.png) |
| VLAN 10 ACL | check if the ACLs are working | Unauthorized traffic blocked | Success | [Before VLan-10 ACL](screenshots/verification/Acls/Vlan10/Before-ACLs/PC2-ping-PC3-PC5-Syslog-Tftp.png) [After VLan-10 ACL](screenshots/verification/Acls/Vlan10/After-ACLs/PC2-ping-PC3-PC5-Syslog-Tftp.png)  |
| VLAN 30 ACL | check if the ACLs are working | Unauthorized traffic blocked | Success | [Before VLan-30 ACL](screenshots/verification/Acls/Vlan30/Before-ACLs/PC5-Ping-PC3-PC7-Syslog.png) [After VLan-30 ACL](screenshots/verification/Acls/Vlan30/After-ACLs/PC5-Ping-PC3-PC7-Syslog.png)  |
| VLAN 40 ACL | check if the ACLs are working | Unauthorized traffic blocked | Success | [Before VLan-40 ACL](screenshots/verification/Acls/Vlan40/Before-ACLs/PC7-Ping-PC3-PC5-Syslog.png) [After VLan-40 ACL](screenshots/verification/Acls/Vlan40/After-ACLs/PC7-Ping-PC3-PC5-Syslog.png) |
| VTY ACL | Only Admin vlan 20 can manage devices | Admin vlan authorized | Success | [VTY ACL](screenshots/verification/SSH-RADIUS/SW50-ACL-VTY.png) |
| Port Security | Unauthorized MAC triggers violation | Port in Err-Disabled |Success | [Err-Disabled](screenshots/verification/Port-Security/SW-10-fa02-err-disabled.png) |
| DHCP Snooping | Binding Table | Legitimate DHCP clients learned | Success | [Binding Table](screenshots/verification/DHCP-Snooping-DAI/SW20-Show-Ip-Dhcp-Snooping-Binding.png) |
| DAI | Legitimate ARP inspected and forwarded | ARP traffic forwarded | Success | [Arp Inspection interfaces](screenshots/verification/DHCP-Snooping-DAI/SW40-Show-Ip-Arp-inspection-Interfaces.png) |

---

## Troubleshooting














