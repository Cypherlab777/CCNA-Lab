# Lab 9 Entreprise Network

## Overview


Ce projet combine plusieurs technologies Cisco vues durant mon apprentissage afin de les faire fonctionner ensemble dans un même environnement.


L'infrastructure simule une entreprise composée de deux sites :


* Un Headquarters (HQ) utilisant une architecture 3-Tier.
* Une Branch Office utilisant une architecture Collapsed Core.
* Le plan d'adressage de l'entreprise est basé sur le réseau privé 172.16.0.0/16, découpé en plusieurs sous-réseaux répartis entre le siège et la succursale.


Ce lab est une sorte de checkpoint me permettant de consolider mes connaissances et de les mettre en pratique dans un environnement plus réaliste.


Technologies mises en œuvre :


* Architecture 3-Tier
* Architecture Collapsed Core
* VLAN Segmentation
* Inter-VLAN Routing
* HSRP
* RPVST+
* OSPF
* DHCP Relay
* Layer 2 Hardening


---


## Objectives



* Concevoir un réseau d'entreprise réaliste en utilisant plusieurs architectures afin de comparer les bénéfices de chacune d'entre elles.
* Mettre en pratique des technologies telles que HSRP et RPVST+ et les aligner afin d'obtenir des chemins cohérents et optimaux, tout en permettant une circulation fluide du trafic.
* Mettre en place un serveur DHCP central utilisé par l'ensemble du réseau via DHCP Relay.
* Offrir de la redondance aux passerelles par défaut des deux sites en configurant HSRP.
* Mettre en place le routage dynamique avec OSPF.
* Mettre en application les bonnes pratiques de sécurité à l'aide du Layer 2 Hardening.
* Manipuler les coûts OSPF afin d'observer le comportement d'ECMP et d'influencer les chemins empruntés par certains VLANs.



---



## Topology Overview



Le réseau d'entreprise est composé d'un Headquarters et d'une Branch Office connectés via un lien WAN (Lease Line).



Le réseau du Headquarters employant beaucoup de monde, j'ai décidé d'utiliser une architecture 3-Tier.



La Branch Office quant à elle utilise une architecture 2-Tier Collapsed Core. Au vu de sa taille, il m'a semblé plus judicieux de fusionner les couches Core et Distribution afin de simplifier l'infrastructure et de réduire les coûts.



La redondance des passerelles par défaut est assurée par HSRP, tandis que RPVST+ contrôle les liens afin d'éviter les boucles Layer 2.



J'ai décidé de déployer un serveur DHCP sur le site principal afin de fournir des adresses IP aux différents VLANs du Headquarters via DHCP Relay configuré sur les passerelles.



Pour la Branch Office, j'ai opté pour un DHCP directement configuré sur le routeur. Cela m'a permis de mettre en pratique une seconde méthode DHCP souvent utilisée sur des sites plus petits.



L'adressage a également été pensé de manière à offrir une marge de croissance de 100 % sur les deux sites afin d'anticiper les besoins futurs.



![Topology](screenshots/Topology/Topology.png)



---



## Network Design & Addressing



---

### HQ — 3-Tier Architecture

---



#### VLAN Design



| Vlan | Name | Network | Gateway SW-D1 | Gateway SW-D2 |
|---|---|---|---|---|
| 10 | Users | 172.16.0.0/21 | 172.16.7.254 | 172.16.7.253 |
| 20 | R&D | 172.16.8.0/22 | 172.16.11.254 | 172.16.11.253 |
| 30 | RH | 172.16.12.0/23 | 172.16.13.254 | 172.16.13.253 |
| 40 | IT | 172.16.14.0/24 | 172.16.14.254 | 172.16.14.253 |
| 50 | Server | 172.16.15.0/25 | 172.16.15.126 | 172.16.15.125 |
| 60 | DR | 172.16.16.0/26 | 172.16.16.62 | 172.16.16.61 |


Le serveur DHCP est situé dans le VLAN 50 et utilise une adresse IP statique : 172.16.15.10.

---

#### HSRP Design


| Vlan | Active | Standby | Virtual Gateway |
|---|---|---|---|
| 10 | SW-D1 | SW-D2 | 172.16.0.1 |
| 20 | SW-D2 | SW-D1 | 172.16.8.1 |
| 30 | SW-D1 | SW-D2 | 172.16.12.1 |
| 40 | SW-D2 | SW-D1 | 172.16.14.1 |
| 50 | SW-D1 | SW-D2 | 172.16.15.1 |
| 60 | SW-D2 | SW-D1 | 172.16.16.1 |

VLAN 10, 30 et 50 utilisent SW-D1 comme passerelle active HSRP et comme Root Bridge RPVST+.

VLAN 20, 40 et 60 utilisent SW-D2 comme passerelle active HSRP et comme Root Bridge RPVST+.


Cette répartition permet d'équilibrer la charge entre les deux switches de distribution tout en maintenant un alignement cohérent entre HSRP et RPVST+.


---


#### Routing Design



| Network | Device | Interface | IP Address |
|---|---|---|---|
| 10.0.0.0/30 | SW-D1 | Po1 | 10.0.0.1 |
| 10.0.0.0/30 | SW-C1 | Po1 | 10.0.0.2 |
| 10.0.0.4/30 | SW-D1 | Fa0/7 | 10.0.0.5 |
| 10.0.0.4/30 | SW-C2 | Gi1/0/3 | 10.0.0.6 |
| 10.0.0.8/30 | SW-D2 | Po1 | 10.0.0.10 |
| 10.0.0.8/30 | SW-C2 | Po1 | 10.0.0.9 |
| 10.0.0.12/30 | SW-D2 | Fa0/7 | 10.0.0.13 |
| 10.0.0.12/30 | SW-C1 | Gi1/0/3 | 10.0.0.14 |
| 10.0.0.16/30 | SW-C1 | Gi1/0/6 | 10.0.0.17 |
| 10.0.0.16/30 | R1-Edge | G0/0 | 10.0.0.18 |
| 10.0.0.20/30 | SW-C2 | Po1 | 10.0.0.21 |
| 10.0.0.20/30 | R1-Edge | G0/1 | 10.0.0.22 |

---

#### Devices Running-Config


| Device | Config |
|---|---|
| SW-A1 | [SW-A1.txt](configs/Headquarters/SW-A1.txt) |
| SW-A2 | [SW-A2.txt](configs/Headquarters/SW-A2.txt) |
| SW-A3 | [SW-A3.txt](configs/Headquarters/SW-A3.txt) |
| SW-A4 | [SW-A4.txt](configs/Headquarters/SW-A4.txt) |
| SW-A5 | [SW-A5.txt](configs/Headquarters/SW-A5.txt) |
| SW-A6 | [SW-A6.txt](configs/Headquarters/SW-A6.txt) |
| SW-D1 | [SW-D1.txt](configs/Headquarters/SW-D1.txt) |
| SW-D2 | [SW-D2.txt](configs/Headquarters/SW-D2.txt) |
| SW-C1 | [SW-C1.txt](configs/Headquarters/SW-C1.txt) |
| SW-C2 | [SW-C2.txt](configs/Headquarters/SW-C2.txt) |
| R1-Edge | [R1-Edge.txt](configs/Headquarters/R1-Edge.txt) |


---


### Branch — 2-Tier Collapsed Core Architecture

---

#### VLAN Design


| Vlan | Name | Network | Gateway SW-CD | Gateway SW-CD2 |
|---|---|---|---|---|
| 10 | Users | 172.16.128.0/22 | 172.16.131.254 | 172.16.131.253 |
| 30 | RH | 172.16.132.0/24 | 172.16.132.254 | 172.16.132.253 |
| 40 | IT | 172.16.133.0/25 | 172.16.133.126 | 172.16.133.125 |
| 60 | DR | 172.16.133.128/26 | 172.16.133.190 | 172.16.133.189 |

---

#### HSRP Design


| Vlan | Active | Standby | Virtual Gateway |
|---|---|---|---|
| 10 | SW-CD | SW-CD2 | 172.16.128.1 |
| 30 | SW-CD | SW-CD2 | 172.16.132.1 |
| 40 | SW-CD2 | SW-CD | 172.16.133.1 |
| 60 | SW-CD2 | SW-CD | 172.16.133.129 |

---

#### Routing Design


| Network | Device | Interface | IP Address |
|---|---|---|---|
| 10.0.1.0/30 | SW-CD | G1/0/5 | 10.0.1.1 |
| 10.0.1.0/30 | R2-Edge | G0/0 | 10.0.1.2 |
| 10.0.1.4/30 | SW-CD2 | G1/0/5 | 10.0.1.5 |
| 10.0.1.4/30 | R2-Edge | G0/1 | 10.0.1.6 |
| 10.0.1.8/30 | SW-CD | G1/0/6 | 10.0.1.9 |
| 10.0.1.8/30 | SW-CD2 | G1/0/6 | 10.0.1.10 |
#### Devices Running-Config


| Device | Config |
|---|---|
| SW-A01 | [SW-A01.txt](configs/SW-A01.txt) |
| SW-A02 | [SW-A02.txt](configs/SW-A02.txt) |
| SW-A03 | [SW-A03.txt](configs/SW-A03.txt) |
| SW-A04 | [SW-A04.txt](configs/SW-A04.txt) |
| SW-CD | [SW-CD.txt](configs/SW-CD.txt) |
| SW-CD2 | [SW-CD2.txt](configs/SW-CD2.txt) |
| R2-Edge | [R2-Edge.txt](configs/R2-Edge.txt) |


---

## Security & Hardening



* Tous les ports inutilisés ont été placés dans le VLAN 999 et administrativement shutdown.
* Activation de PortFast par défaut sur les switches d'accès.
* Activation de BPDU Guard par défaut sur les switches d'accès.
* Le VLAN natif des trunks est également configuré sur le VLAN 999 afin d'éviter l'utilisation du VLAN 1.
* Le VLAN 999 n'est pas utilisé pour le trafic utilisateur.
* Restriction des VLANs autorisés sur les trunks.
* Désactivation de DTP.
* Activation de passive-interface sur les SVI des switches de distribution.

---

## Headquarters Verification & Testing

---

### VLAN Verification


![Vlan verification](screenshots/Verification/Headquarters/Vlan/SW-A1-Show-Vlan-Brief-SW-A1.png)



Les VLANs ont été correctement créés sur les switchs d'accès


---


### Trunk Verification



![Trunk verification](screenshots/Verification/Headquarters/Trunk/SW-A2-Show-Interface-Trunk.png)



Les liens trunk sont opérationnels et transportent uniquement les VLANs autorisés



---



### RPVST+ Verification



#### SW-D1



![RPVST+ verification](screenshots/Verification/Headquarters/RPVST+/SW-D1-Show-Spanning-Tree-Vlan10-Vlan.30.png)



SW-D1 est Root Bridge pour les VLANs 10, 30 et 50 afin d'être aligné avec HSRP.



#### SW-D2



![RPVST+ verification](screenshots/Verification/Headquarters/RPVST+/SW-D2-Show-Spanning-Tree-Vlan20-Vlan.40.png)



SW-D2 est Root Bridge pour les VLANs 20, 40 et 60 afin d'être aligné avec HSRP.

---



### HSRP Verification



#### SW-D1



![HSRP verification](screenshots/Verification/Headquarters/HSRP/SW-D1-Show-Standby-brief.png)



SW-D1 est actif pour les VLANs 10, 30 et 50 tandis que SW-D2 reste en état Standby.



#### SW-D2



![HSRP verification](screenshots/Verification/Headquarters/HSRP/SW-D2-Show-Standby-brief.png)



SW-D2 est actif pour les VLANs 20, 40 et 60 tandis que SW-D1 reste en état Standby.


---


### OSPF Convergence


![OSPF verification](screenshots/Verification/Headquarters/OSPF/SW-D1-Show-IP-Ospf-Neighbor.png)


Les adjacences OSPF ont été établies avec tous les voisins attendus.


---

### End-to-End Connectivity

---

#### Internal Connectivity


![inside Connectivity verification](screenshots/Verification/Headquarters/Inside-Connectivity/Vlan10-Ping-Vlan60.png)



![inside Connectivity verification](screenshots/Verification/Headquarters/Inside-Connectivity/Vlan20-Ping-Vlan40.png)



La communication entre plusieurs VLANs a été validée afin de confirmer le bon fonctionnement du routage inter-VLAN à l'intérieur du HQ



#### External Connectivity



![Outside Connectivity verification](screenshots/Verification/Headquarters/Outside-Connectivity/3-tier-Vlan20-Ping-2-Tier-Vlan30.png)



La connectivité entre le Headquarter et la Branch a été vérifiée avec succès à travers le WAN.

---

#### HSRP / RPVST+ Path Validation



![Tracert Path verification](screenshots/Verification/Headquarters/Tracert-Path/PC1-Tracert-R1-Edge-Path-SWD1.png)



![Tracert Path verification](screenshots/Verification/Headquarters/Tracert-Path/PC2-Tracert-R1-Edge-Path-SW-D2.png)



Les résultats du traceroute confirment que le trafic emprunte les chemins prévus grâce à l'alignement entre HSRP et RPVST+.


---

## Branch Office Verification & Testing

---

### VLAN Verification


![Vlan verification](screenshots/Verification/Branch-Office/Vlan/SW-A01-Show-Vlan-Brief.png)


Les VLANs ont été correctement créés sur les switchs d'accès


---


### Trunk Verification


![Trunk verification](screenshots/Verification/Branch-Office/Trunk/SW-CD2-Show-Interface-Trunk.png)


Les liens trunk sont opérationnels et transportent uniquement les VLANs autorisés


---


### RPVST+ Verification


#### SW-CD


![RPVST+ verification](screenshots/Verification/Branch-Office/RPVST+/SW-CD-Show-Spanning-Tree-Vlan10-Vlan.30.png)

SW-CD est Root Bridge pour les VLANs 10 et 30 afin d'être aligné avec HSRP.



#### SW-CD2



![RPVST+ verification](screenshots/Verification/Branch-Office/RPVST+/SW-CD-Show-Spanning-Tree-Vlan40-Vlan.60.png)



SW-CD2 est Root Bridge pour les VLANs 40 et 60 afin d'être aligné avec HSRP.


---



### HSRP Verification



#### SW-CD



![HSRP verification](screenshots/Verification/Branch-Office/HSRP/SW-CD-Show-Standby-brief.png)



SW-CD est actif pour les VLANs 10 et 30 tandis que SW-CD2 reste en état Standby.



#### SW-CD2



![HSRP verification](screenshots/Verification/Branch-Office/HSRP/SW-CD2-Show-Standby-brief.png)



SW-CD2 est actif pour les VLANs 40 et 60 tandis que SW-CD reste en état Standby.



---



### OSPF Convergence



![OSPF verification](screenshots/Verification/Branch-Office/OSPF/SW-CD-Show-IP-Ospf-Neighbor.png)



Les adjacences OSPF ont été établies avec tous les voisins attendus.



---

### DHCP Verification

![DHCP verification](screenshots/Verification/Branch-Office/DHCP/SW-CD-Show-IP-Dhcp-Pool.png)


Le routeur SW-CD distribue correctement les adresses IP aux clients des VLANs de la Branch Office.

---

### End-to-End Connectivity



#### Internal Connectivity



![Internal Connectivity verification](screenshots/Verification/Branch-Office/Internal-Connectivity/Vlan10-Ping-Vlan40.png)



La communication entre les VLANs 10 et 40 a été validée afin de confirmer le bon fonctionnement du routage inter-VLAN à l'intérieur de la Branch.



---



#### External Connectivity



![External Connectivity verification](screenshots/Verification/Branch-Office/External-Connectivity/2-tier-Vlan10-Ping-3-Tier-Vlan20.png)



La connectivité entre la Branch et le Headquarter a été vérifiée avec succès à travers le WAN.



---



#### HSRP / RPVST+ Path Validation



![Tracert Path verification](screenshots/Verification/Branch-Office/Tracert-Path/PC11-Tracert-R2-Edge-Path-SW-CD.png)



![Tracert Path verification](screenshots/Verification/Branch-Office/Tracert-Path/PC12-Tracert-R2-Edge-Path-SW-CD.png)



Les résultats du traceroute confirment que le trafic emprunte les chemins prévus grâce à l'alignement entre HSRP et RPVST+.



---


## DORA Analysis


### Limitations de Packet Tracer concernant le DORA



Packet Tracer n'affiche pas toutes les options DHCP en détail. Ces options contiennent normalement des informations comme :


* Le lease time
* Le serveur DNS
* Le type de message DHCP : Discover, Offer, Request, ACK
* Le Server Identifier
* La Requested IP Address
* Etc.


L'adresse du DHCP Relay devrait également apparaître dans le champ Relay Agent Address / giaddr, mais Packet Tracer ne l'affiche pas correctement dans ce lab.



L'observation des paquets DHCP serait plus complète avec Wireshark sous GNS3. Malheureusement je n'ai pas la possibilité d'utiliser ce dernier.



Dans ce lab, l'analyse est donc limitée aux informations visibles.

---

### Discover

![Discover verification](screenshots/DHCP-Analysis/DHCP-Discover/DHCP-Doscover(1).png)


![Discover verification](screenshots/DHCP-Analysis/DHCP-Discover/DHCP-Discover(2).png)


Nous pouvons observer sur ces captures le paquet Discover du processus DORA.

Ce paquet sert a découvrir si il y a un serveur DHCP.


Dans la partie ethernet et ip nous avons :


SRC IP : 0.0.0.0 = Le pc n'a pas encore d'adresse ip

DEST ADDR : FFFF.FFFF.FFFF = Paquet envoyé à l'adresse broadcast Layer 2

DEST IP : 255.255.255.255 = paquet envoyé à l'adresse de broadcast Layer 3


Le paquet est envoyé en broadcast pour découvrir si un serveur DHCP existe


Dans la partie UDP nous avons :


SOURCE PORT : 68 = Ce qui correspond au port client

DEST PORT : 67 = Ce qui correspond au port serveur


Dans la partie DHCP :



OP:0x01 = BOOTREQUEST

CLIENT HARDWARE : 0000.0CB7.37DC = Adresse MAC du pc


---


### Offer


![Offer verification](screenshots/DHCP-Analysis/DHCP-Offer/DHCP-Offer-from-server-to-Relay-GW-Vlan60(1).png)


![Offer verification](screenshots/DHCP-Analysis/DHCP-Offer/DHCP-Offer-from-server-to-Relay-GW-Vlan60(2).png)


Nous pouvons observer sur ces captures le paquet Offer du processus DORA.

Ce paquet sert à envoyer une proposition d'adresse IP au client depuis le serveur DHCP.


Dans la partie ethernet et iP nous avons :


SRC IP : 172.16.15.10 = L'adresse IP du Serveur DHCP

DEST ADDR = 000.0C07.AC32 = Adresse MAC du DHCP Relay

DEST IP : 172.16.16.61 = Adresse IP du DHCP Relay


Dans la partie DHCP :


OP:0x02 = BOOTREPLY

YOUR CLIENT ADDRESS 172.16.16.10 = Adresse offerte par le serveur DHCP au client

SERVER ADDRESS : 172.16.15.10 = Adresse IP du Serveur DHCP

CLIENT HARDWARE : 0000.0CB7.37DC = Adresse MAC du pc


---


### Request



![Request verification](screenshots/DHCP-Analysis/DHCP-Request/DHCP-Request-From-pc-to-serveur(1).png)



![Request verification](screenshots/DHCP-Analysis/DHCP-Request/DHCP-Request-From-pc-to-serveur(2).png)



Nous pouvons observer sur ces captures le paquet Request du processus DORA

Le paquet est envoyé en broadcast afin d'informer tous les serveurs DHCP ayant répondu que le client accepte cette offre et rejette les éventuelles autres propositions.


Dans la partie ethernet et IP nous avons :


SRC IP : 0.0.0.0 = Le pc n'a pas encore d'adresse IP

DEST ADDR : FFFF.FFFF.FFFF = Paquet envoyé à l'adresse broadcast Layer 2

DEST IP : 255.255.255.255 = paquet envoyé à l'adresse de broadcast Layer 3


Dans la partie DHCP :


OP:0x01 = BOOTREQUEST

CLIENT HARDWARE : 0000.0CB7.37DC = Adresse MAC du PC


Dans un vrai paquet DHCP, l'adresse demandée par le client serait visible dans les options DHCP, notamment avec l'option Requested IP Address. Packet Tracer ne l'affiche pas clairement ici.

---

### Ack



![Ack verification](screenshots/DHCP-Analysis/DHCP-Ack/DHCP-ACK-from-server-to-PC(1).png)



![Ack verification](screenshots/DHCP-Analysis/DHCP-Ack/DHCP-ACK-from-server-to-PC(2).png)


Nous pouvons observer sur ces captures le paquet Ack du processus DORA

Ce paquet sert à confirmer officiellement l'attribution de l'adresse IP demandée par le client.



Dans la partie ethernet et IP nous avons :



SRC IP : 172.16.15.10 = L'adresse IP du Serveur DHCP

DEST ADDR = 000.0C07.AC32 = Adresse MAC du  DHCP Relay

DEST IP : 172.16.16.61 = Adresse IP du DHCP Relay



Dans la partie DHCP :


OP:0x02 = BOOTREPLY

YOUR CLIENT ADDRESS 172.16.16.10 = Adresse offerte par le serveur DHCP au client

SERVER ADDRESS : 172.16.15.10 = Adresse IP du Serveur DHCP

CLIENT HARDWARE : 0000.0CB7.37DC = Adresse MAC du pc

---

## Troubleshooting

---

### Issue #1 — OSPF Adjacency Missing (SW-D2 ↔ SW-C2)

Lors d'un test de connectivité entre PC9 et SW-C2, je me suis rendu compte que le paquet empruntait une route étrange.

Chemin attendu :

* PC9 - SW-A5 - SW-D2 - SW-C2

Chemin constaté :

* PC9 - SW-A5 - SW-D2 - SW-A4 - SW-D1 - SW-C2

Je me suis demandé pourquoi le paquet faisait ce long détour, ce qui me semblait être un comportement assez étrange.

J'ai donc vérifié les adjacences OSPF et je me suis rendu compte que sur le Port-Channel 1, aucune adjacence n'était établie et qu'aucun paquet Hello n'était reçu malgré OSPF activé.

J'ai ensuite vérifié la configuration sur SW-C2 et j'ai remarqué qu'aucune adresse IP n'était configurée sur le Port-Channel 1.

J'ai donc décidé de configurer l'adresse manquante.

Une fois l'adresse configurée, à ma grande surprise, un message indiquant un chevauchement d'adresses (« overlap with Gi1/0/2 ») est apparu.

Gi1/0/2 étant un port membre du Port-Channel 1, j'ai alors vérifié sa configuration et constaté que l'adresse IP y était configurée.

J'ai donc supprimé cette adresse avec la commande no ip address sur Gi1/0/2 puis reconfiguré cette même adresse sur Po1.

Une fois cela fait, le message indiquant l'établissement d'une nouvelle adjacence OSPF s'est affiché à l'écran.

Le trafic a immédiatement recommencé à emprunter le chemin attendu.

Problème résolu !

---

Symptoms 

* Chemin étrange constaté.
* Aucune adjacence OSPF présente sur Po1.

---

Root Cause

* L'adresse IP était configurée sur une interface physique membre de l'EtherChannel au lieu du Port-Channel.

---

Solution

* Suppression de l'adresse IP sur Gi1/0/2.
* Configuration de l'adresse IP sur le Port-Channel 1.
* Vérification de l'établissement de l'adjacence OSPF.

---

### Issue #2 — ECMP Path Sélection Not Optimal

---

Lors d'un test de connectivité entre PC1 et SW-C1, je me suis rendu compte que le paquet empruntait une route étrange, un peu comme dans le problème numéro 1.

Chemin attendu :

SW-A1 - SW-D1 - SW-C1 - SW-D1 - SW-A1 - PC1

Chemin constaté :

SW-A1 - SW-D1 - SW-C1 - SW-D2 - SW-A4 - SW-D1 - SW-A1 - PC1

Je me suis de nouveau demandé pourquoi le paquet, à partir de SW-D2, ne passait pas directement par SW-A1.

J'ai vérifié le spanning-tree du VLAN 10 sur SW-D2 et je me suis rendu compte que le port Fa0/1 était dans l'état BLK. J'avais donc déjà un début d'explication.

Mais cela ne réglait pas mon problème d'origine : pourquoi le paquet n'empruntait-il pas le chemin qui me semblait le plus logique ?

J'ai alors vérifié la table de routage et j'ai remarqué qu'il y avait deux chemins disponibles vers le VLAN 10. Je me suis donc rendu compte qu'ECMP faisait simplement son travail, c'est-à-dire utiliser plusieurs chemins ayant le même coût.

À mon sens, le chemin pouvait être optimisé. J'ai donc décidé d'augmenter le coût OSPF du lien entre SW-C1 et SW-D2 à 5.

Cette modification a eu pour effet de supprimer le second chemin de coût égal vers cette destination et de forcer le trafic à emprunter le chemin souhaité.

Problème résolu !

---

Symptoms

* Chemin inattendu constaté lors du test de connectivité.

---

Root Cause

* Deux chemins de coût égal étaient présents dans la table de routage, ce qui entraînait l'utilisation 
d'ECMP.

---

Solution

* Augmentation du coût OSPF sur le lien SW-C1 ↔ SW-D2 afin de privilégier le chemin souhaité.

---

## Skills Gained

* Conception d'une architecture 3-Tier et Collapsed Core.
* Création d'un plan d'adressage VLSM pour plusieurs sites.
* Configuration et validation de HSRP.
* Configuration et optimisation de RPVST+.
* Mise en œuvre du routage dynamique avec OSPF.
* Manipulation des coûts OSPF afin d'influencer les chemins de routage.
* Configuration d'un serveur DHCP central avec DHCP Relay.
* Configuration d'un DHCP local sur routeur.
* Mise en place d'un EtherChannel LACP.
* Application de mesures de Layer 2 Hardening.
* Validation du fonctionnement du réseau à l'aide de commandes de vérification Cisco.
* Analyse et résolution de problèmes réseau.

---

## Key Concepts Learned

* L'importance d'aligner HSRP et RPVST+ afin d'obtenir des chemins de trafic cohérents.
* Le fonctionnement d'ECMP et son impact sur la sélection des chemins.
* L'influence des coûts OSPF sur les décisions de routage.
* La différence entre DHCP Relay et DHCP local sur routeur.
* Le rôle du Root Bridge dans le comportement de RPVST+.
* Le fonctionnement des adjacences OSPF sur les interfaces Layer 3 et les EtherChannels.
* L'importance de vérifier les interfaces physiques et logiques lors d'un dépannage.
* Les limites de Packet Tracer pour l'analyse détaillée des paquets DHCP.
* L'intérêt d'une approche méthodique lors du troubleshooting réseau.

---
