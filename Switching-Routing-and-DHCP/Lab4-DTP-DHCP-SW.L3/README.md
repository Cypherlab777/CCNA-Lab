# Lab4 : Dynamic Trunk protocol + DHCP Switch L3

## Objective 

Build a small network using multiple VLANs, DTP trunk negotiation, inter-VLAN routing and DHCP services on a multilayer switch.
---

## Topology 

- 4 switch 2960
- 4 PCs
- 1 Switch L3 3560
---

## Network configuration 

SW.SALES : 
fa0/2 = switchport mode access
fa0/1 = dynamic auto

SW.IT : 
fa0/2 = switchport mode access
fa0/1 = dynamic auto

SW.RH : 
fa0/2 = switchport mode access 
fa0/1 = switchport mode trunk

SW.USERS : 
fa0/2 = switchport mode access
fa0/1 = switchport mode dynamic desirable

SW.L3
fa0/1 = dynamic auto 
fa0/2 = dynamic auto
Fa0/4 = switchport mode dynamic desirable
Fa0/3 = switchport mode dynamic desirable
---

## Configurations 
```bash

SW.SALES : 
enable
conf t

hostname SW.SALES

vlan 10
name SALES

interface fa0/2
switchport mode access
switchport access vlan 10
do show vlan brief

show interface fa0/1 sw
```

```bash

SW.IT : 
enable 
conf t

hostname SW.IT

vlan 20
name IT

interface fa0/2
switchport mode access
switchport access vlan 20
do show vlan brief

show interface fa0/1 sw
```
```bash

SW.RH
enable 
conf t

hostname SW.RH

vlan 30 
name RH

interface fa0/2
switchport mode access
switchport access vlan 30
do show vlan brief

interface fa0/1
switchport mode trunk
do show interface fa0/1 sw
```
```bash

SW.USERS
enable
conf t

hostname SW.USERS

vlan 40
name USERS

interface fa0/2
switchport mode access
switchport access vlan 40
do show vlan brief

interface fa0/1
switchport mode dynamic desirable
do show interface fa0/1 sw
```
```bash

SW.L3 : 
enable 
conf t

ip routing

interface vlan 10
ip address 192.168.10.1 255.255.255.0
no shutdown


ip dhcp excluded-address 192.168.10.1 192.168.10.20
ip DHCP pool SALES
network 192.168.10.0 255.255.255.0
default-router 192.168.10.1
dns-server 8.8.8.8

show ip dhcp pool
show interface fa0/4 sw

interface fa0/4
switchport mode dynamic desirable
do show interface fa0/4 sw
```
The same configuration pattern was applied for VLANs 20, 30 and 40.
---

## Vérification 
```bash

show vlan brief
show interface trunk
show interface fa0/x sw
show ip interface brief
show ip route
show ip dhcp pool
ping
```

## Observations

I initially expected to configure dot1q encapsulation manually on the multilayer switch, but the trunk links worked without it.

After verification with:
show interfaces switchport

the switch was already operating with:
Operational Trunking Encapsulation: dot1q

It appears Packet Tracer automatically uses 802.1Q encapsulation on this switch model.
----

## Skills gained

- Configured VLANs and trunk links using DTP
- Configured inter-VLAN routing on a multilayer switch
- Configured DHCP pools for multiple VLANs
- Verified trunk negotiation and routing operations
- Learned that modern Cisco switches primarily use 802.1Q instead of ISL
---