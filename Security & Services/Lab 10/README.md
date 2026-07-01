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

Le vlan 20 Admin a un access totale a tous les vlans ainsi qu'a tous les equipements pour le management

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

The following credentials are used only for this Packet Tracer lab.
They are intentionally simple and must not be reused in a real environment.

| Access Type | Username | Password / Secret |
|---|---|---|
| RADIUS SSH user | admin | cisco |
| Local fallback user | localadmin | localpass |
| Enable secret | N/A | admin123 |

### ACLs Design note

Pour les besoins du lab j'ai configurer les ACLs comme suit : 

1. Ce qui est permit l'est explicitement 
2. Ce qui est restreint l'est explicitement egalement 
3. le permit final autorise le reste des communications
4. Les serveurs ne doivent pas initier librement des connections vers les vlans ou internet, ils ne peuvent fournir que les services autorises  
5. le filtrage principal est applique cote VLAN clients, car les ACL classiques ne sont pas stateful.

---

## Security Features & Verification 

### Extended ACLs

Les ACLs etendue sont appliquer au plus pres de la source, ce type d'ACL etant beaucoup plus precise, cela ne risque pas d'autoriser ou de refuser du traffic non desire 
Les ACLs sont principalement pour des restrictions inter-vlan mais aussi pour les sercices

![R3 Show Access-lists](screenshots/verification/Acls/R3-Show-Access-Lists.png)

### SSH / RADIUS

SSH est utilise pour le management a distance
Un serveur RADIUS authentifie egalement les connections aux equipements, en cas de down le login local reste disponible 

![RADIUS Server](screenshots/verification/SSH-RADIUS/RADIUS-Server.png)

### VTY Acces Control

Une ACLs a ete appliquer sur chaque VTY de chaque equipement pour n'autoriser que le Vlan 20 a s'y connecter.

![VTY ACL](screenshots/verification/SSH-RADIUS/SW50-ACL-VTY.png)

### Syslog

Un Serveur Syslog a ete mis en place afin de recuperer et de centraliser tous les evenements important.
A noter que sur packet tracer il n'y a que l'option de log debugging qui etait disponible

![Syslog Server](screenshots/verification/Syslog/Syslog.png)

### TFTP Backup

Un Serveur TFTP a ete mis en place afin d'y sauvegarder les configurations des equipements, cela facilite la gestion mais aussi offre plus de securite en cas de probleme 

![Tftp Server](screenshots/verification/Tftp/Tftp.png)

### Port security

La fonctionnalite port-security est activer sur les ports access utilisateurs afin d'eviter tout rogue device non autorise

![SW10 Port-security](screenshots/verification/Port-Security/SW10-Port-Security-fa02-fa03.png)

### DHCP Snooping

DHCP snooping a ete appliquer sur tous les switchs afin de prevenir un rogue DHCP

Seuls les ports de confiance, comme les uplinks vers les routeurs, ont été configurés en trusted.

La limitation des paquets par secondes a 10 a ete appliquer sur TOUS les ports des Switchs afin d'eviter qu'un port utilisateur n'envoyent un trop grand nombre de requete DHCP

![Binding Table](screenshots/verification/DHCP-Snooping-DAI/SW20-Show-Ip-Dhcp-Snooping-Binding.png)

### Dynamic Arp Inspection

DAI a été activé sur les VLANs utilisateurs. Les uplinks vers les routeurs ont été configurés en trusted, tandis que les ports utilisateurs sont restés en untrusted.

![Arp Inspection interfaces](screenshots/verification/DHCP-Snooping-DAI/SW40-Show-Ip-Arp-inspection-Interfaces.png)

### Remark

Dynamic ARP Inspection se basant sur la DHCP Snooping binding table, j'ai d'abord activé DHCP Snooping sur les switches et attendu que les tables se remplissent afin d'éviter que DAI ne bloque du trafic légitime.

L’option 82 de DHCP Snooping a été désactivée, car dans Packet Tracer, lorsqu’elle est active avec DHCP Relay, elle peut casser le DHCP.

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

### Probleme 1 : Management de SW10 impossible depuis le Vlan 20 Admin

Lors d'un test, il m'était impossible de me connecter au SW10 via SSH depuis le VLAN Admin.

Après maintes vérifications, tout semblait correct, jusqu'à ce que j'inspecte l'ACL VLan-10.

En regardant de plus près, je me suis aperçu qu'en fait, l'ACL, qui est stateless, faisait bien son travail et bloquait la connexion retour : SW10 → VLAN 20

Cause : l'ACL bloquait le chemin retour vers le VLAN Admin.

Solution : implémenter une ACE established autorisant les connexions retour TCP.

Cette solution a également été implémentée dans les ACLs VLan-30 et VLan-40.

---

### Probleme 2 : Le Server Syslog ne recoit plus aucun logs depuis les SW10 SW30 et SW40

Lors de la vérification du système de logs, je me suis aperçu que les logs ne se centralisaient plus sur le serveur Syslog.

Après investigation, il s'est révélé que les ACLs faisaient une nouvelle fois bien leur travail en bloquant les connexions de ces VLANs vers le serveur Syslog.

Il me fallait donc mettre en place une exception pour quand même autoriser ces switches à envoyer leurs logs au serveur.

Cause : les ACLs bloquaient l'accès au serveur Syslog.

Solution : mettre en place une ACE qui autorise les connexions UDP sur le port 514 vers le serveur Syslog.

---

### Probleme 3 : Aucune connectivite depuis pc2

Lors d'un test de connectivité, je me suis aperçu que PC2 n'arrivait plus à joindre aucun réseau distant.

J'ai effectué un tracert pour comprendre où cela bloquait et, à ma grande surprise, le trafic s'arrêtait à la passerelle par défaut.

Après vérification avec ipconfig, il m'est apparu que le PC n'avait pas de passerelle par défaut configurée, alors que la configuration avait été faite via DHCP.

J'ai donc vérifié si le serveur DHCP avait bien une passerelle par défaut configurée dans le pool, ce qui était le cas.

J'ai ensuite exécuté sur PC2 les commandes suivantes :

ipconfig /release
ipconfig /renew

Après cela, PC2 a bien reçu la passerelle par défaut.

Cause : aucune passerelle par défaut configurée sur PC2 via DHCP, probablement à cause d'un bug Packet Tracer ou d'un renouvellement DHCP incomplet.

Solution : renouveler le bail DHCP avec :

ipconfig /release
ipconfig /renew.

---

### Probleme 4 : DHCP failed après la configuration de DHCP snooping

Lors d'une vérification après l'implémentation de DHCP Snooping, je me suis rendu compte que le DHCP ne fonctionnait plus.

Après investigation, je n'ai pas pu trouver le problème seul. C'est à ce moment-là que j'ai demandé à l'IA de m'aider. Elle a rapidement identifié le problème.

J'avais oublié de mettre le port vers R1 en trust, ce qui a eu pour effet de bloquer le trafic DHCP légitime.

Cause : le port vers R1 n'était pas configuré en trust.

Solution : mettre le port vers R1 en trust afin de laisser passer le trafic DHCP légitime.

Cette solution a ensuite été appliquée à tous les autres switches concernés.

---

## Skills Gained


- Built and applied extended ACLs for inter-VLAN filtering.
- Understood ACL order and the impact of the final permit statement.
- Configured SSH-based device management.
- Integrated RADIUS authentication with local fallback.
- Restricted VTY access to an Admin VLAN.
- Configured Syslog for centralized event logging.
- Used TFTP to back up network device configurations.
- Implemented Port Security with sticky MAC learning.
- Tested Port Security violations and err-disabled behavior.
- Configured DHCP Snooping and trusted/untrusted ports.
- Configured Dynamic ARP Inspection based on DHCP Snooping bindings.
- Validated security controls using Cisco verification commands.
- Hardening L2

## Key Concepts Learned

- Extended ACLs should be placed close to the source.
- ACL entries are processed from top to bottom.
- The final `permit ip ... any` does not override previous deny entries.
- Cisco IOS ACLs are stateless.
- DHCP traffic needs special ACL handling because clients initially use `0.0.0.0`.
- Management traffic may require exceptions when management IPs are in user VLANs.
- RADIUS uses a shared secret between the device and server, separate from the user password.
- Local fallback is important to avoid being locked out.
- DHCP Snooping is required before DAI can validate ARP traffic dynamically.
- Only uplinks should normally be trusted for DHCP Snooping and DAI.
- Port Security in shutdown mode places the interface into err-disabled state.
- Syslog helps confirm and document security events.

---












