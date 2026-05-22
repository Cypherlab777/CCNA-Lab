# Lab5 : CDP - VOICE VLAN

## Objective

creation of a small network to see how CDP & LLDP work  
creation of two VLANs and one voice VLAN to analyze CDP behavior with a sniffer

## Topology

- 2 Switch 2960
- 2 IP phones
- 3 Sniffer
- 2 PCs
- 1 Switch 3650

## Network Configuration

- VLAN10 = PC0 : 192.168.10.2/24 
- VLAN20 = PC1 : 192.168.20.2/24
- VLAN30 = VOICE VLAN

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
show cdp neighbors
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
show cdp neighbors
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
show ip dhcp binding
```

## Verification

```bash
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
```

Sniffer frame analysis

---

## troubleshooting

problem1 : no connectivity from pc0 to pc1  
cause : IP Phone on pc1 side was turned off  
solution : turn on the IP Phone on pc1 side

problem2 : no IP on the IP Phones  
cause : no DHCP on SW.L3  
solution : create a VOICE30 pool on SW.L3

problem3 : IP received on IP Phone 1 but not on IP Phone 2  
cause : when I connected the sniffer I connected the PC output to the switch  
solution : correctly reconnect the IP Phone (pc-pc / sw-sw)

problem4 : DHCP configuration was not working, IP Phones were not receiving IP addresses  
cause : TFTP server not specified  
solution : add `option 150 ip 192.168.30.1` in the DHCP pool VOICE30 on SW.L3

---

# Notes & Observation 

- On SW1 I can see SW.L3, I can also see that it is a 3650 with a holdtime.  
On SW2 I can see the same thing.

- On sniffer 0 I noticed that in the first CDP frames there is no VLAN TAG and that the destination address is 0100.0CCC.CCCC which is the proprietary multicast address used by CDP.

- I also noticed that in the latest CDP frames the TPID field 0x8100 corresponds to the 802.1Q tag. I observed that these CDP frames on the trunk were tagged VLAN 1 while the CDP frames sent to the phone were untagged.

- When I configured DHCP voice vlan on SW.L3 I noticed that it was not working.  
After some research I realized that I had to provide a "configuration file" to the IP Phone which is normally located on a TFTP/CUCM server.

But under Packet Tracer there is no real TFTP server so I had to use the Voice VLAN gateway 192.168.30.1 with option 150.

---

## Skills gained

- I learned that the IP Phone learns the Voice VLAN via CDP & once the Voice VLAN is learned, the phone then tags its voice traffic in 802.1Q.

- I learned that the switch sends untagged CDP frames to the phone. After learning the Voice VLAN via CDP, the phone then sends its voice traffic tagged in 802.1Q.

- I learned that CDP frames can be transported via the native VLAN (untagged) or via 802.1Q (tagged). This depends on the type of link used (access or trunk).

- I learned that to configure a DHCP voice vlan it was necessary to specify the TFTP server.

- I learned that the hello timer = 60 seconds.

- I learned that the Hold-timer = 180 seconds (3x hello timer = 180sec).

- I observed that the holdtime decreases then resets after receiving a new CDP advertisement.

- I learned that Cisco uses a proprietary multicast address for CDP : 0100.0CCC.CCCC.

- I learned that the field TPID:0x8100 corresponds to 802.1Q & the field TCI:0x0001 corresponds to the VLAN id.

---


