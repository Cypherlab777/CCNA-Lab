# Lab 9 Enterprise Network Design

## Overview

This project combines several Cisco technologies covered during my studies and brings them together within the same environment.

The infrastructure simulates a company composed of two sites:

* A Headquarters (HQ) using a 3-Tier architecture.
* A Branch Office using a Collapsed Core architecture.
* The company's addressing plan is based on the private network 172.16.0.0/16, divided into multiple subnets distributed between the headquarters and the branch office.

This lab serves as a checkpoint allowing me to consolidate my knowledge and apply it in a more realistic environment.

Technologies implemented:

* 3-Tier Architecture
* Collapsed Core Architecture
* VLAN Segmentation
* Inter-VLAN Routing
* HSRP
* RPVST+
* OSPF
* DHCP Relay
* Layer 2 Hardening

---

## Objectives

* Design a realistic enterprise network using multiple architectures in order to compare the benefits of each approach.
* Implement technologies such as HSRP and RPVST+ and align them to achieve consistent and optimized traffic paths while maintaining smooth traffic flow.
* Deploy a centralized DHCP server used by the entire network through DHCP Relay.
* Provide default gateway redundancy for both sites by configuring HSRP.
* Implement dynamic routing using OSPF.
* Apply security best practices through Layer 2 Hardening.
* Manipulate OSPF costs in order to observe ECMP behavior and influence the paths used by specific VLANs.

---

## Topology Overview

The enterprise network consists of a Headquarters and a Branch Office connected through a WAN link (Leased Line).

Since the Headquarters hosts a large number of users, I decided to use a 3-Tier architecture.

The Branch Office uses a 2-Tier Collapsed Core architecture. Due to its smaller size, I felt it was more appropriate to merge the Core and Distribution layers in order to simplify the infrastructure and reduce costs.

Default gateway redundancy is provided by HSRP, while RPVST+ controls the links to prevent Layer 2 loops.

I decided to deploy a DHCP server at the main site in order to provide IP addresses to the different Headquarters VLANs through DHCP Relay configured on the gateways.

For the Branch Office, I chose to use DHCP directly configured on the router. This allowed me to practice a second DHCP deployment method that is commonly used on smaller sites.

The addressing plan was also designed to provide a 100% growth margin on both sites in order to anticipate future requirements.



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


The DHCP server is located in VLAN 50 and uses the following static IP address: 172.16.15.10.

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


VLANs 10, 30, and 50 use SW-D1 as their active HSRP gateway and RPVST+ Root Bridge.

VLANs 20, 40, and 60 use SW-D2 as their active HSRP gateway and RPVST+ Root Bridge.

This distribution helps balance the load between the two distribution switches while maintaining consistent alignment between HSRP and RPVST+.

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
| SW-A1 | [SW-A1.txt](configs/Headquarters/SW-A/SW-A1.txt) |
| SW-A2 | [SW-A2.txt](configs/Headquarters/SW-A/SW-A2.txt) |
| SW-A3 | [SW-A3.txt](configs/Headquarters/SW-A/SW-A3.txt) |
| SW-A4 | [SW-A4.txt](configs/Headquarters/SW-A/SW-A4.txt) |
| SW-A5 | [SW-A5.txt](configs/Headquarters/SW-A/SW-A5.txt) |
| SW-A6 | [SW-A6.txt](configs/Headquarters/SW-A/SW-A6.txt) |
| SW-D1 | [SW-D1.txt](configs/Headquarters/SW-D/SW-D1.txt) |
| SW-D2 | [SW-D2.txt](configs/Headquarters/SW-D/SW-D2.txt) |
| SW-C1 | [SW-C1.txt](configs/Headquarters/SW-C/SW-C1.txt) |
| SW-C2 | [SW-C2.txt](configs/Headquarters/SW-C/SW-C2.txt) |
| R1-Edge | [R1-Edge.txt](configs/Headquarters/R1-Edge/R1-Edge.txt) |


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

---

#### Devices Running-Config


| Device | Config |
|---|---|
| SW-A01 | [SW-A01.txt](configs/Branch-Office/SW-A/SW-A01.txt) |
| SW-A02 | [SW-A02.txt](configs/Branch-Office/SW-A/SW-A02.txt) |
| SW-A03 | [SW-A03.txt](configs/Branch-Office/SW-A/SW-A03.txt) |
| SW-A04 | [SW-A04.txt](configs/Branch-Office/SW-A/SW-A04.txt) |
| SW-CD | [SW-CD.txt](configs/Branch-Office/SW-CD/SW-CD.txt) |
| SW-CD2 | [SW-CD2.txt](configs/Branch-Office/SW-CD/SW-CD2.txt) |
| R2-Edge | [R2-Edge.txt](configs/Branch-Office/R2-Edge/R2-Edge.txt) |


---

## Security & Hardening

* All unused ports have been placed in VLAN 999 and administratively shut down.
* PortFast has been enabled by default on access switches.
* BPDU Guard has been enabled by default on access switches.
* The native VLAN on trunk links is also configured as VLAN 999 to avoid using VLAN 1.
* VLAN 999 is not used for user traffic.
* Allowed VLANs have been restricted on trunk links.
* DTP has been disabled.
* Passive interfaces have been configured on the SVIs of the distribution switches.

---

## Headquarters Verification & Testing

---

### VLAN Verification


![Vlan verification](screenshots/Verification/Headquarters/Vlan/SW-A1-Show-Vlan-Brief-SW-A1.png)


The VLANs have been successfully created on the access switches.

### Trunk Verification

![Trunk verification](screenshots/Verification/Headquarters/Trunk/SW-A2-Show-Interface-Trunk.png)

The trunk links are operational and carry only the allowed VLANs.

---

### RPVST+ Verification

#### SW-D1

![RPVST+ verification](screenshots/Verification/Headquarters/RPVST+/SW-D1-Show-Spanning-Tree-Vlan10-Vlan.30.png)

SW-D1 is the Root Bridge for VLANs 10, 30, and 50 in order to align with HSRP.

#### SW-D2

![RPVST+ verification](screenshots/Verification/Headquarters/RPVST+/SW-D2-Show-Spanning-Tree-Vlan20-Vlan.40.png)

SW-D2 is the Root Bridge for VLANs 20, 40, and 60 in order to align with HSRP.

---

### HSRP Verification

#### SW-D1

![HSRP verification](screenshots/Verification/Headquarters/HSRP/SW-D1-Show-Standby-brief.png)

SW-D1 is active for VLANs 10, 30, and 50 while SW-D2 remains in the Standby state.

#### SW-D2

![HSRP verification](screenshots/Verification/Headquarters/HSRP/SW-D2-Show-Standby-brief.png)

SW-D2 is active for VLANs 20, 40, and 60 while SW-D1 remains in the Standby state.

---

### OSPF Convergence

![OSPF verification](screenshots/Verification/Headquarters/OSPF/SW-D1-Show-IP-Ospf-Neighbor.png)

OSPF adjacencies have been successfully established with all expected neighbors.

---

### End-to-End Connectivity

---

#### Internal Connectivity

![inside Connectivity verification](screenshots/Verification/Headquarters/Inside-Connectivity/Vlan10-Ping-Vlan60.png)

![inside Connectivity verification](screenshots/Verification/Headquarters/Inside-Connectivity/Vlan20-Ping-Vlan40.png)

Communication between multiple VLANs has been successfully validated, confirming proper inter-VLAN routing within the Headquarters.

#### External Connectivity

![Outside Connectivity verification](screenshots/Verification/Headquarters/Outside-Connectivity/3-tier-Vlan20-Ping-2-Tier-Vlan30.png)

Connectivity between the Headquarters and the Branch Office has been successfully verified across the WAN.

---

#### HSRP / RPVST+ Path Validation

![Tracert Path verification](screenshots/Verification/Headquarters/Tracert-Path/PC1-Tracert-R1-Edge-Path-SWD1.png)

![Tracert Path verification](screenshots/Verification/Headquarters/Tracert-Path/PC2-Tracert-R1-Edge-Path-SW-D2.png)

The traceroute results confirm that traffic follows the expected paths thanks to the alignment between HSRP and RPVST+.

---

## Branch Office Verification & Testing

---

### VLAN Verification

![Vlan verification](screenshots/Verification/Branch-Office/Vlan/SW-A01-Show-Vlan-Brief.png)

The VLANs have been successfully created on the access switches.

---

### Trunk Verification

![Trunk verification](screenshots/Verification/Branch-Office/Trunk/SW-CD2-Show-Interface-Trunk.png)

The trunk links are operational and carry only the allowed VLANs.

---

### RPVST+ Verification

#### SW-CD

![RPVST+ verification](screenshots/Verification/Branch-Office/RPVST+/SW-CD-Show-Spanning-Tree-Vlan10-Vlan.30.png)

SW-CD is the Root Bridge for VLANs 10 and 30 in order to align with HSRP.

#### SW-CD2

![RPVST+ verification](screenshots/Verification/Branch-Office/RPVST+/SW-CD-Show-Spanning-Tree-Vlan40-Vlan.60.png)

SW-CD2 is the Root Bridge for VLANs 40 and 60 in order to align with HSRP.

---

### HSRP Verification

#### SW-CD

![HSRP verification](screenshots/Verification/Branch-Office/HSRP/SW-CD-Show-Standby-brief.png)

SW-CD is active for VLANs 10 and 30 while SW-CD2 remains in the Standby state.

#### SW-CD2

![HSRP verification](screenshots/Verification/Branch-Office/HSRP/SW-CD2-Show-Standby-brief.png)

SW-CD2 is active for VLANs 40 and 60 while SW-CD remains in the Standby state.

---

### OSPF Convergence

![OSPF verification](screenshots/Verification/Branch-Office/OSPF/SW-CD-Show-IP-Ospf-Neighbor.png)

OSPF adjacencies have been successfully established with all expected neighbors.

---

### DHCP Verification

![DHCP verification](screenshots/Verification/Branch-Office/DHCP/SW-CD-Show-IP-Dhcp-Pool.png)

The SW-CD router is correctly assigning IP addresses to clients in the Branch Office VLANs.

---

### End-to-End Connectivity

#### Internal Connectivity

![Internal Connectivity verification](screenshots/Verification/Branch-Office/Internal-Connectivity/Vlan10-Ping-Vlan40.png)

Communication between VLANs 10 and 40 has been successfully validated, confirming proper inter-VLAN routing within the Branch Office.

---

#### External Connectivity

![External Connectivity verification](screenshots/Verification/Branch-Office/External-Connectivity/2-tier-Vlan10-Ping-3-Tier-Vlan20.png)

Connectivity between the Branch Office and the Headquarters has been successfully verified across the WAN.

---

#### HSRP / RPVST+ Path Validation

![Tracert Path verification](screenshots/Verification/Branch-Office/Tracert-Path/PC11-Tracert-R2-Edge-Path-SW-CD.png)

![Tracert Path verification](screenshots/Verification/Branch-Office/Tracert-Path/PC12-Tracert-R2-Edge-Path-SW-CD.png)

The traceroute results confirm that traffic follows the expected paths thanks to the alignment between HSRP and RPVST+.

---

## DORA Analysis

### Packet Tracer Limitations Regarding DORA

Packet Tracer does not display all DHCP options in detail. These options normally contain information such as:

* Lease time
* DNS server
* DHCP message type: Discover, Offer, Request, ACK
* Server Identifier
* Requested IP Address
* Etc.

The DHCP Relay address should also appear in the Relay Agent Address / giaddr field, but Packet Tracer does not display it correctly in this lab.

Observing DHCP packets would be more complete using Wireshark with GNS3. Unfortunately, I do not have the possibility to use it.

In this lab, the analysis is therefore limited to the information that is visible.

---

### Discover

![Discover verification](screenshots/DHCP-Analysis/DHCP-Discover/DHCP-Doscover\(1\).png)

![Discover verification](screenshots/DHCP-Analysis/DHCP-Discover/DHCP-Discover\(2\).png)

We can observe the Discover packet of the DORA process in these captures.

This packet is used to discover whether a DHCP server is available.

In the Ethernet and IP sections we have:

SRC IP : 0.0.0.0 = The PC does not yet have an IP address

DEST ADDR : FFFF.FFFF.FFFF = Packet sent to the Layer 2 broadcast address

DEST IP : 255.255.255.255 = Packet sent to the Layer 3 broadcast address

The packet is sent as a broadcast to discover whether a DHCP server exists.

In the UDP section we have:

SOURCE PORT : 68 = Corresponds to the client port

DEST PORT : 67 = Corresponds to the server port

In the DHCP section:

OP:0x01 = BOOTREQUEST

CLIENT HARDWARE : 0000.0CB7.37DC = PC MAC address

---

### Offer

![Offer verification](screenshots/DHCP-Analysis/DHCP-Offer/DHCP-Offer-from-server-to-Relay-GW-Vlan60\(1\).png)

![Offer verification](screenshots/DHCP-Analysis/DHCP-Offer/DHCP-Offer-from-server-to-Relay-GW-Vlan60\(2\).png)

We can observe the Offer packet of the DORA process in these captures.

This packet is used to send an IP address proposal from the DHCP server to the client.

In the Ethernet and IP sections we have:

SRC IP : 172.16.15.10 = DHCP Server IP address

DEST ADDR = 0000.0C07.AC32 = DHCP Relay MAC address

DEST IP : 172.16.16.61 = DHCP Relay IP address

In the DHCP section:

OP:0x02 = BOOTREPLY

YOUR CLIENT ADDRESS 172.16.16.10 = IP address offered by the DHCP server to the client

SERVER ADDRESS : 172.16.15.10 = DHCP Server IP address

CLIENT HARDWARE : 0000.0CB7.37DC = PC MAC address

---

### Request

![Request verification](screenshots/DHCP-Analysis/DHCP-Request/DHCP-Request-From-pc-to-serveur\(1\).png)

![Request verification](screenshots/DHCP-Analysis/DHCP-Request/DHCP-Request-From-pc-to-serveur\(2\).png)

We can observe the Request packet of the DORA process in these captures.

The packet is sent as a broadcast to inform all DHCP servers that responded that the client accepts this offer and rejects any other proposals.

In the Ethernet and IP sections we have:

SRC IP : 0.0.0.0 = The PC does not yet have an IP address

DEST ADDR : FFFF.FFFF.FFFF = Packet sent to the Layer 2 broadcast address

DEST IP : 255.255.255.255 = Packet sent to the Layer 3 broadcast address

In the DHCP section:

OP:0x01 = BOOTREQUEST

CLIENT HARDWARE : 0000.0CB7.37DC = PC MAC address

In a real DHCP packet, the address requested by the client would be visible in the DHCP options, notably through the Requested IP Address option. Packet Tracer does not display it clearly here.

---

### Ack

![Ack verification](screenshots/DHCP-Analysis/DHCP-Ack/DHCP-ACK-from-server-to-PC\(1\).png)

![Ack verification](screenshots/DHCP-Analysis/DHCP-Ack/DHCP-ACK-from-server-to-PC\(2\).png)

We can observe the Ack packet of the DORA process in these captures.

This packet is used to officially confirm the assignment of the IP address requested by the client.

In the Ethernet and IP sections we have:

SRC IP : 172.16.15.10 = DHCP Server IP address

DEST ADDR = 0000.0C07.AC32 = DHCP Relay MAC address

DEST IP : 172.16.16.61 = DHCP Relay IP address

In the DHCP section:

OP:0x02 = BOOTREPLY

YOUR CLIENT ADDRESS 172.16.16.10 = IP address offered by the DHCP server to the client

SERVER ADDRESS : 172.16.15.10 = DHCP Server IP address

CLIENT HARDWARE : 0000.0CB7.37DC = PC MAC address

---

## Troubleshooting

---

### Issue #1 — OSPF Adjacency Missing (SW-D2 ↔ SW-C2)

During a connectivity test between PC9 and SW-C2, I noticed that the packet was taking an unusual path.

Expected path:

* PC9 - SW-A5 - SW-D2 - SW-C2

Observed path:

* PC9 - SW-A5 - SW-D2 - SW-A4 - SW-D1 - SW-C2

I wondered why the packet was taking such a long detour, which seemed like a rather strange behavior.

I checked the OSPF adjacencies and noticed that no adjacency was established on Port-Channel 1 and that no Hello packets were being received even though OSPF was enabled.

I then checked the configuration on SW-C2 and noticed that no IP address was configured on Port-Channel 1.

I decided to configure the missing address.

Once the address was configured, to my surprise, a message indicating an address overlap ("overlap with Gi1/0/2") appeared.

Since Gi1/0/2 was a member port of Port-Channel 1, I checked its configuration and discovered that the IP address was configured there.

I removed the address using the `no ip address` command on Gi1/0/2 and then configured the same address on Po1.

Once this was done, a message indicating that a new OSPF adjacency had been established appeared on the screen.

Traffic immediately started using the expected path again.

Problem solved!

---

Symptoms

* Unusual path observed.
* No OSPF adjacency present on Po1.

---

Root Cause

* The IP address was configured on a physical interface that was a member of the EtherChannel instead of on the Port-Channel interface.

---

Solution

* Removed the IP address from Gi1/0/2.
* Configured the IP address on Port-Channel 1.
* Verified the establishment of the OSPF adjacency.

---

### Issue #2 — ECMP Path Selection Not Optimal

---

During a connectivity test between PC1 and SW-C1, I noticed that the packet was taking an unusual path, similar to Issue #1.

Expected path:

SW-A1 - SW-D1 - SW-C1 - SW-D1 - SW-A1 - PC1

Observed path:

SW-A1 - SW-D1 - SW-C1 - SW-D2 - SW-A4 - SW-D1 - SW-A1 - PC1

Once again, I wondered why the packet, after reaching SW-D2, was not going directly through SW-A1.

I checked the spanning-tree status for VLAN 10 on SW-D2 and noticed that port Fa0/1 was in the BLK state. This already gave me part of the explanation.

However, this did not answer my original question: why was the packet not using what seemed to be the most logical path?

I then checked the routing table and noticed that there were two available paths to VLAN 10. I realized that ECMP was simply doing its job, which is to use multiple paths with the same cost.

In my opinion, the path could be optimized. I therefore decided to increase the OSPF cost of the link between SW-C1 and SW-D2 to 5.

This change removed the second equal-cost path to the destination and forced the traffic to use the desired path.

Problem solved!

---

Symptoms

* Unexpected path observed during the connectivity test.

---

Root Cause

* Two equal-cost paths were present in the routing table, resulting in ECMP being used.

---

Solution

* Increased the OSPF cost on the SW-C1 ↔ SW-D2 link in order to prefer the desired path.

---

## Skills Gained

* Designed a 3-Tier and Collapsed Core architecture.
* Created a VLSM addressing plan for multiple sites.
* Configured and validated HSRP.
* Configured and optimized RPVST+.
* Implemented dynamic routing with OSPF.
* Manipulated OSPF costs to influence routing paths.
* Configured a centralized DHCP server with DHCP Relay.
* Configured local DHCP on a router.
* Implemented an EtherChannel using LACP.
* Applied Layer 2 Hardening measures.
* Validated network operation using Cisco verification commands.
* Analyzed and resolved network issues.

---

## Key Concepts Learned

* The importance of aligning HSRP and RPVST+ to achieve consistent traffic paths.
* How ECMP works and its impact on path selection.
* The influence of OSPF costs on routing decisions.
* The difference between DHCP Relay and local DHCP on a router.
* The role of the Root Bridge in RPVST+ behavior.
* How OSPF adjacencies operate on Layer 3 interfaces and EtherChannels.
* The importance of verifying both physical and logical interfaces during troubleshooting.
* The limitations of Packet Tracer for detailed DHCP packet analysis.
* The value of a methodical approach when troubleshooting network issues.

---
