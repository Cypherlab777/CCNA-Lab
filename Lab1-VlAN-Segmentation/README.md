#Lab 1 - VLAN segmentation

## Objective 

Basic vlan segmentation
Secure unused switch port for security


- VLAN 10 = Management
- VLAN 20 = RH
- VLAN 99 = SAFE
---

## Topology 

- 4 pcs
- 1 switch
--- 

## Network information 

	Management  			RH

     pc1 = 192.168.10.2 	 pc3 = 192.168.20.2
     pc2 = 192.168.10.3		 pc4 = 192.168.20.3
---

## Configuration 


```bash
enable 
conf t

vlan 10
name MANAGEMENT 

vlan 20
name RH

vlan 99 
name SAFE

interface range fa0/1-2
switchport mode access
switchport access vlan 10
do show vlan brief

interface range fa0/3-4
switchport mode access
switchport access vlan 20
do show vlan brief

interface range fa0/5-24
switchport mode access
switchport access vlan 99
shutdown

interface range g0/1-2
switchport mode access
switchport access vlan 99
shutdown
do show vlan brief
```

## Vérification 

```bash
show vlan brief
show ip interface brief
show interfaces status
PC1 ping PC2
PC3 ping PC4
```

## Troubleshooting

---

## Resolution 

---

## Skill learned 

- Création & configuration of basic vlan segmentation
- Hardening port for security
