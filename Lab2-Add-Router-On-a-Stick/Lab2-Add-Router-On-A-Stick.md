#Lab 2 - Add Router-On-A-Stick

## Objective

Add a router for inter-vlan communication 
---

## Topology

- 1 Router 2911
---

## Network information

VLAN 10 MANAGEMENT Gateway = 192.168.10.1
VLAN 20 RH Gateway = 192.168.20.1

VLAN 10 interface = Gig0/0.10
VLAN 20 interface = Gig0/0.20
---

## Configuration

```bash

Router :

enable 
conf t

interface g0/0.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0
exit

interface g0/0.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0
exit

interface g0/0
no shutdown
end

show ip interface brief

switch : 

enable 
conf t
interface g0/1
switchport mode trunk
no shutdown
end

show interface trunk
```
## Vérification

```bash

router : 
show ip interface brief
show ip route

switch : 
show interface trunk

PC : ping 
```

## Troubleshooting

Problem 1: Subinterface up but protocol down
Cause: G0/1 administratively down and trunk not configured
Solution: Configure trunk on G0/1 and enable the interface on the switch

Problem 2 : PC1 cannot ping PC4
Cause : Missing default Gateway on PCs
Solution : Configure default Gateway on PCs

## Skills gained 
## Skills gained

-Création of router sub-interfaces for inter-VLAN routing
-Configuration of default gateways on PCs
-Troubleshooting trunk and Gateway issues
