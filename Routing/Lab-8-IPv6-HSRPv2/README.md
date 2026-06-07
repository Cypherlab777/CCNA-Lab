# Lab 8 IPv6 - HSRPv2

---

## Objective

---

Build an end-to-end IPv6 network using:

 - IPv6 static routing
 - SLAAC (EUI-64)
 - Neighbor Discovery Protocol (NDP)
 - Router Solicitation (RS)
 - Router Advertisement (RA)
 - HSRPv2 gateway redundancy
 - IPv6 failover testing

---

## Topology 

![Topology](Screenshots/Topology/Topology.png)

---

## Network Configuration

### Router Addressing Plan

| Router | Interface | Vlan | IPv6 | Prefix | Rôle |
|---|---|---|---|---|---|
| R1 | G0/0/0.10 | 10 | FD00:10:0:10::1 | /64 | Real Gateway | 
| R1 | G0/0/0.20 | 20 | FD00:10:0:20::1 | /64 | Real Gateway|
| R1 | G0/0/1 | N/A | 2001:1::1 | /64 | N/A |
| R2 | G0/0/0.10 | 10 | FD00:10:0:10::2 | /64 | Real Gateway |
| R2 | G0/0/0.20 | 20 | FD00:10:0:20::2 | /64 | Real Gateway |
| R2 | G0/0/1 | N/A | 2001:2::1 | /64 | N/A |
| R3 | G0/0/0 | N/A | 2001:1::2 | /64 | N/A |
| R3 | G0/0/1 | N/A | 2001:2::2 | /64 | N/A |
| R3 | Se0/2/0 | N/A | 2001:3::1 | /64 | N/A |
| R4 | Se0/2/0 | N/A | 2001:3::2 | /64 | N/A |
| R4 | Se0/0/0 | N/A | 2001:5::1 | /64 | N/A |

| Vlan | HSRP Group | Virtual Gateway |
|---|---|---|
| 10 |    10 | FE80::5:73FF:FEA0:10 |
| 20 |    20 | FE80::5:73FF:FEA0:20 |

### PCs Addressing Plan

| Device | Vlan | IPv6 | Prefix | Default Gateway |
|---|---|---|---|---|
| PC1 | 10 | EUI-64 (SLAAC) | /64 | FE80::5:73FF:FEA0:10 |
| PC2 | 20 | EUI-64 (SLAAC) | /64 | FE80::5:73FF:FEA0:20 |
| DC | N/A | 2001:5::10 | /64 | 2001:5::1 |

### Device Configurations

| Router | Configuration            |
| ------ | ------------------------ |
| R1     | [R1.txt](Configs/R1.txt) |
| R2     | [R2.txt](Configs/R2.txt) |
| R3     | [R3.txt](Configs/R3.txt) |
| R4     | [R4.txt](Configs/R4.txt) |
| SW1    | [SW1.txt](Configs/SW1.txt) |

---

## Vérification

![R1 Routing table](Screenshots/Vérification/R1-Show-IPv6-Route.png)

 - Verification of R1's routing table

---

![R1 Neighbors table](Screenshots/Vérification/R1-Show-IPv6-Neighbor.png)

 - Verification of R1's IPv6 neighbor table (NDP) 

---

![R2 Standby](Screenshots/Vérification/R2-Show-Standby.png)

 - Verification of the HSRPv2 configuration on R2
 - Observation of R2 in Standby mode for both groups

---

![R3 Routing table](Screenshots/Vérification/R3-Show-IPv6-Route.png)

 - Verification of R3's routing table
 - Observation of the IPv6 host route (/128)

---


## NDP/SLAAC Analysis

### Router Solicitation

![Router Sollicitation](Screenshots/NDP-SCLAAC/PC2-RS-0x85-All-Router.png)

 - We can observe the packet sent by PC2 containing the Type 133 (0x85) Router Solicitation message in order to obtain the information required for IPv6 configuration through SLAAC.
 - We can also observe that the packet is sent to the FF02::2 address, which corresponds to the All Routers multicast address, because PC2 does not know R1's MAC address.

---

### Router Advertisement 

![Router Advertisement](Screenshots/NDP-SCLAAC/R1-RA-0x86-All-Nodes.png)

 - We can observe the packet sent by R1 containing the Type 134 (0x86) Router Advertisement message in response to the Router Solicitation sent by PC2.
 - This packet contains the network prefix as well as R1's gateway information (which is the HSRPv2 virtual gateway).
 - We can also observe that the packet is sent to the FF02::1 address, which corresponds to the All Nodes multicast address, because Router Advertisements are intended for all hosts present on the segment.
 - The "M" and "O" fields are empty, which confirms that this is a SLAAC configuration packet (M=0 ; O=0).

### EUI-64/SLAAC

![EUI-64/SLAAC PC1](Screenshots/NDP-SCLAAC/PC1-EUI-64-Successful.png)

 - EUI-64/SLAAC configuration on PC1 successful
 - IPv6 ULA: FD00:10:0:10:203:E4FF:FE94:86B6/64
 - HSRPv2 virtual IP = FE80::5:73FF:FEA0:10

![EUI-64/SLAAC PC2](Screenshots/NDP-SCLAAC/PC2-EUI-64-Successful.png)

 - EUI-64/SLAAC configuration on PC2 successful
 - HSRPv2 virtual IP = FE80::5:73FF:FEA0:20
 - IPv6 ULA: FD00:10:0:20:20A:F3FF:FE3B:22B6/64

---


## Initial Configuration Verification

![PC1 Tracert](Screenshots/Initial-config/PC1-Tracert-DC.png)

 - We can observe the normal packet path from PC1, going through R1, which is the Active router.

---

![PC2 Tracert](Screenshots/Initial-config/PC2-Tracert-DC.png)

 - We can observe the normal packet path from PC2, going through R1, which is the Active router.

---

## Simulating a Failure on R1

![R2 failover](Screenshots/R1-Down-config/R2-failover.png)

 - A failure occurred on R1.
 - R2, which was previously in Standby mode, took over and became Active.
 - Normally, with tracking and preemption, the failover should have occurred automatically when interface G0/0/1 on R1 went down.
 - Since HSRPv2 and some features are not correctly simulated, and tracking is not available on the 4331 routers, I had to completely disable the interfaces on R1 for the failover to occur properly.
 - I also had to manually change the priorities.

---

### NDP Cache Issue

![Bad NDP entry](Screenshots/R1-Down-config/PC2-Still-MAC-R1.png)

 - As we can see, PC2 had kept the association between the HSRP virtual address and R1's MAC address in memory:
FE80::5:73FF:FEA0:20 (HSRP Gateway) ---> 0060.3EBC.A801 (R1 MAC)
 - Whereas the entry should normally have been:
FE80::5:73FF:FEA0:20 (HSRP Gateway) ---> 0007.EC48.8C01 (R2 MAC)

---

### Failover Verification

![Pc2 Tracert new path](Screenshots/R1-Down-config/PC2-Tracert-DC-Failover-Path.png)

 - After rebooting PC2 due to the cache issue, pings started working correctly again.
 - We can observe the new path taken by packets from PC2, now going through R2 following the failover.
---

### R1 Returns

![R1 back](Screenshots/R1-Down-config/R1-Back.png)

 - After the intervention on R1, R1 resumes its role as the Active router.
 - Normally, R1 should have automatically regained its role thanks to preemption.
 - Once again, I had to perform the change manually due to a Packet Tracer issue.
---


## Troubleshooting

Issue 1 : 

 - Initially, I wanted to manually configure the IPv6 addresses of the PCs as well as their virtual HSRP gateway. However, in Packet Tracer, HSRP for IPv6 only provided automatic virtual address configuration. Therefore, I was unable to manually define a global virtual gateway such as FD00:10:0:10::1.

 - Solution : I configured the PCs using SLAAC so that they could automatically learn the HSRP virtual gateway through Router Advertisements.

---

Issue 2 : 

 - PC1 could not ping the DC.
 - Cause: IPv6 unicast-routing was not enabled on R3.
 - Solution: Enable IPv6 unicast-routing on R3.

---

Issue 3 :

 - During my initial configuration, I used 2911 routers. Once everything was configured, I encountered several issues :

 - Preemption remained disabled even after being enabled.
 - R2 did not become Active despite tracking being enabled.
 - Various routing issues.

I was therefore unable to observe and document the behavior I wanted accurately.

 - Cause : Packet Tracer limitations and bugs that do not simulate HSRPv2 and some related features very well.

 - Solution: I changed the equipment and switched to 4331 routers. Although these routers do not support tracking, HSRPv2 works much better with these models.

For the purpose of the failover simulation, I also manually modified the priorities to alternately place the routers into Active and Standby states.

---

Issue 4 :

 - After simulating a failure on R1, I noticed that my pings no longer worked at all, and tracert failed as well.
 - Cause: The PCs had kept R1's old entries in their NDP cache. The virtual gateway was still associated with R1's MAC address even though R2 had become the Active router.

In a real environment, the newly Active router should normally announce the change so that hosts update their NDP cache. However, Packet Tracer does not seem to simulate this behavior correctly.

 - Solution: I rebooted the PCs. The NDP cache was reset and connectivity was restored.

---

## Skills Gained

 - EUI-64/SLAAC Configuration
 - NDP Packet Analysis (RS/RA)
 - IPv6 Router-on-a-Stick
 - IPv6 HSRPv2
 - IPv6 Static Routing
 - IPv6 Host Route (/128)
 - IPv6 Default Route
 - IPv6/HSRPv2 Troubleshooting
 - Identifying Packet Tracer Limitations (HSRPv2 IPv6, NDP Cache)
