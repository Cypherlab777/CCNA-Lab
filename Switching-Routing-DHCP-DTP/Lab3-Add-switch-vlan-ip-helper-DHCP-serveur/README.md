#Lab 3 - Add 2 new vlan + serveur DHCP

## Objective 

Add 3 new vlans
 point to point link between router0 and router1
Assign ip and Gateway to each pc via DHCP

- VLAN 30 = SALES
- VALN 40 = IT
- VLAN 50 = SERVEUR
- VLAN 99 = SAFE
- Point to point link
---

## Topology 

- 1 Switch 2960
- 1 router 2911
- 1 serveur
- 4 PCs
---

## Network configuration 
```
Router0 : 
interface g0/1 = 10.0.0.1/30

router1 :
interface g0/0 = 10.0.0.2/30 
interface g0/0.30 = 192.168.30.1
interface g0/0.40 = 192.168.40.1
interface g0/0.50 = 192.168.50.1

DHCP server = 192.168.50.100 255.255.255.0
```

## Configuration
```bash

Switch2 : 
enable
conf t

vlan 30
name SALES

vlan 40
name IT

vlan 50
name SERVEUR

vlan 99 
name SAFE

do show vlan brief

interface range fa0/2-3
switchport mode access
switchport access vlan 30

interface range fa0/4-5
switchport mode access
switchport access vlan 40

interface  fa0/6
switchport mode access
switchport access vlan 50

interface fa0/1
switchport mode trunk
do show interface fa0/1 sw

interface range fa0/7-24
switchport mode access
switchport access vlan 99
shutdown

interface range g0/1-2
switchport mode access
switchport access vlan 99
shutdown

show vlan brief

Server : 
Power on

Create pool = VLAN30 
Gateway = 192.168.30.1
start ip address = 192.168.30.10
subnet mask = 255.255.255.0

Create pool = VLAN40
Gateway = 192.168.40.1
start ip address = 192.168.40.10
subnet mask = 255.255.255.0

router1 : 
enable 
conf t

interface g0/1.30
encapsulation dot1Q 30
ip address 192.168.30.1 255.255.255.0

interface g0/1.40
encapsulation dot1Q 40
ip address 192.168.40.1 255.255.255.0

interface g0/1.50
encapsulation dot1Q 50
ip address 192.168.50.1 255.255.255.0

interface g0/1
no shutdown

interface g0/0
ip address 10.0.0.2 255.255.255.252
no shutdown

show ip interface brief
show ip route

interface g0/1.30
ip helper-address 192.168.50.100

interface g0/1.40
ip helper-address 192.168.50.100

interface g0/1.50
ip helper-address 192.168.50.100

ip route

router0 : 
enable 
conf t

interface g0/1
ip address 10.0.0.1 255.255.255.252
no shutdown 

show ip interface brief
show ip route
ping 10.0.0.2

PC5,6,7,8 : 
ipconfig
ipconfig /renew

```

## Vérification 
```bash

show vlan brief
show interface trunk
show interface fa0/1 sw
show ip interface brief
show ip route
ping
```

## Troubleshooting 

```
problem 1 : DHCP on PC5 fail
cause : no ip addresse configured on the DHCP server
Solution : Configure an ip address on DHCP serveur

Problem2 : server can ping vlan 30 et 40 gateway but DHCP still fail on PCs
cause : no ip helper-address configured on the router
Explanation : The `ip helper-address` is necessary because the DORA process's `discover` request is a broadcast and therefore does not traverse routers.
solution : configure ip helper-address on router sub-interfaces

problem3 : PC5 cannot ping pc1 / destination host unreachable
cause : R1 does not know 192.168.10.0 - 192.168.20.0
	R0 does not know 192.168.30.0 - 192.168.40.0 - 192.168.50.0
	Missing static route
solution : Configure static route on both router for each network
```

## Skills gained

- Creation and configuration of a DHCP serveur
- Configuration of Ip helper-address 
- Configuration of Static route
- Configuration of a Point to point router link
- troubleshooting DHCP and Routing issues
---

## Notes

- In the Screenshot, the first ping from PC1 (25% loss) fails due to the ARP resolution.

