#Lab5 : CDP - VOICE VLAN

## Objective

creation d'un petit réseau pour voir comment fonctionne CDP & LLDP
création de deux vlan et d'un voice vlan pour analyser avec un sniffer le comportement de CDP

## Topology

- 2 Switch 2960
- 2 IP phones
- 3 Sniffer
- 2 PCs
- 1 Switch 3650

## Network Configutation

- VLAN10 = PC0 : 192.168.10.2/24 
- VLAN20 = PC1 : 192.168.20.2/24
- VLAN 30 = VOICE VLAN

- SW1:
VLAN10 + VOICE VLAN 30
fa0/1 = switchport mode access
fa0/2 = switchport mode trunk

- SW2
VLAN20 + VOICE VLAN 30
fa0/1 = switchport mode access
fa0/2 = switchport mode trunk

- SW.L3
interface vlan 10 = 192.168.10.1
interface vlan 20 = 192.168.20.1
interface vlan 30 = 192.168.30.1
g1/0/1 = switchport mode trunk
g1/0/2 = switchport mode trunk
---

## Configuration 
```bash

SW1 :
enable
conf t

hostname SW1

vlan 10
vlan 30

interface fa0/1
switchport mode access
switchport access vlan 10
switchport voice vlan 30
do show vlan brief
do show interface fa0/1 switchport

interface fa0/2
switchport mode trunk

show cdp
show cdp Neighbors
```
```bash

SW2 : 
enable 
conf t

hostname SW2

vlan 20
vlan 30

interface fa0/1
switchport mode access
switchport access vlan 20
switchport voice vlan 30
do show vlan brief
do show interface fa0/1 switchport

interface fa0/2
switchport mode trunk

show cdp
show cdp Neighbors
```
```bash

SW.L3
enable 
conf t

hostname SW.L3

ip routing

interface vlan 10
ip address 192.168.10.1 255.255.255.0
no shutdown

interface vlan 20
ip address 192.168.20.1 255.255.255.0
no shutdown

interface vlan 30
ip address 192.168.30.1 255.255.255.0
no shutdown

interface g1/0/1
switchport mode dynamic desirable

interface g1/0/2
switchport mode dynamic desirable

show ip interface brief
show ip route
show interface trunk

ip DHCP pool VOICE30
network 192.168.30.0 255.255.255.0
default-router 192.168.30.1
option 150 ip 192.168.30.1

show ip dhcp pool
show ip DHCP binding
```

## Verification

show vlan brief
show interface trunk
show interface fa0/x switchport
show ip interface brief
show ip route
show ip dhcp pool
show ip dhcp binding
show cdp
show cdp neighbors
show cdp neighbors detail

Analyse des trames dans le sniffer
---

## troubleshooting

problem1 : pas de connectivité de pc0 à pc1  
cause : IP Phone coté pc1 éteint  
solution : allumer le IP Phone pc1

problem2 : pas d'IP sur les IP Phone  
cause : pas de DHCP sur le SW.L3  
solution : créer un pool VOICE30 sur le SW.L3

problem3 : IP reçue sur IP Phone 1 mais pas sur IP Phone 2  
cause : quand j'ai mis le sniffer j'ai connecté la sortie PC sur le switch  
solution : brancher correctement le IP Phone (pc-pc / sw-sw)

problem4 : la configuration DHCP ne fonctionnait pas, les IP Phone ne recevaient pas d'IP  
cause : serveur TFTP non précisé  
solution : ajouter `option 150 ip 192.168.30.1` dans le pool DHCP VOICE30 sur SW.L3
---

# Notes & Observation 

- Sur SW1 je vois SW.L3, je vois également que c'est un 3650 avec un holdtime.  
Sur SW2 je vois la même chose.


- Sur sniffer 0 je remarque que dans les premieres trames CDP il n'y a pas de TAG VLAN et que l'adresse de destination est 0100.0CCC.CCCC qui est l'adresse multicast propriétaire utilisée par CDP.

- Je remarque également que dans les dernieres trames CDP le champ TPID 0x8100 correspond au tag 802.1Q. J’ai observé que ces trames CDP sur le trunk étaient taggées VLAN 1 alors que les trames CDP envoyées vers le téléphone étaient non taggées.


- Quand j'ai configuré le DHCP voice vlan sur SW.L3 j'ai remarqué que cela ne fonctionnait pas.  
Après recherche je me suis rendu compte qu'il fallait donner un "fichier de configuration" à l’IP Phone qui se trouve normalement sur un serveur TFTP/CUCM.  

Mais sous Packet Tracer il n'y a pas de vrai serveur TFTP donc il fallait mettre la gateway du Voice VLAN 192.168.30.1 avec l'option 150.
---

## Skills gained

- j'ai appris que l’IP Phone apprend le Voice VLAN via CDP & une fois le Voice VLAN appris,
le téléphone tag ensuite son trafic voix en 802.1Q.

- j'ai appris que le switch envoie les trames CDP non taggées au téléphone. Après avoir appris
le Voice VLAN via CDP, le téléphone transmet ensuite son trafic voix taggé en 802.1Q.

- J'ai appris que les trames CDP peuvent être transportées via le VLAN natif (non taggé) ou via 802.1Q (taggé). Cela dépend du type de lien utilisé (access ou trunk).

- J'ai appris que pour configurer un DHCP voice vlan il fallait stipuler le serveur TFTP.

- J'ai appris qu'il y a le hello timer = 60 secondes.

- j'ai appris qu'il y a le Hold-timer = 180 secondes (3x hello timer = 180sec).

- j'ai observé que le holdtime diminue puis se réinitialise après réception d'une nouvelle annonce CDP.

- J'ai appris que Cisco utilisait une adresse multicast propriétaire pour CDP : 0100.0CCC.CCCC.

- J'ai appris que le champ TPID:0x8100 correspond au 802.1Q & le champ TCI:0x0001 au VLAN id
---
 




