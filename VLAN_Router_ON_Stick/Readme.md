# Router‑on‑a‑Stick with VLANs – Cisco Packet Tracer

## Project Overview

This project demonstrates **inter‑VLAN routing** using the **Router‑on‑a‑Stick** method.  
A single physical link between a **Layer 2 switch** and a **router** carries traffic for multiple VLANs.  
The router uses **subinterfaces** with 802.1Q encapsulation to route packets between VLANs.

### Network Segments

| Department | VLAN ID | Subnet          | Gateway IP   | Client IPs         |
|------------|---------|----------------|--------------|--------------------|
| Sales      | 10      | 192.168.10.0/24 | 192.168.10.1 | 192.168.10.12, .13 |
| ER         | 20      | 192.168.1.0/24  | 192.168.1.1  | 192.168.1.2, .3    |

> **Note:** The ER department uses `192.168.1.0/24`. This is intentionally different from the Sales subnet, requiring the router to forward traffic between them.

## Topology

- **1 Router** (e.g., Cisco 4321)  
- **1 Layer 2 Switch** (e.g., Cisco 2960)  
- **2 Sales PCs** (VLAN 10) – first two switch ports  
- **2 ER PCs** (VLAN 20) – next two switch ports  

### Port Assignments

| Device          | Switch Port | VLAN | IP Address       |
|----------------|-------------|------|------------------|
| Sales PC1      | Fa0/1       | 10   | 192.168.10.12/24 |
| Sales PC2      | Fa0/2       | 10   | 192.168.10.13/24 |
| ER PC1         | Fa0/3       | 20   | 192.168.1.2/24   |
| ER PC2         | Fa0/4       | 20   | 192.168.1.3/24   |
| Router (trunk) | Gi0/1       | Trunk| N/A              |

Router subinterfaces:
- `G0/0.10` → VLAN 10, IP `192.168.10.1/24`
- `G0/0.20` → VLAN 20, IP `192.168.1.1/24`

## Switch Configuration (Layer 2)

Create VLANs and assign access ports.

```cisco
Switch> enable
Switch# configure terminal

! Create VLANs
Switch(config)# vlan 10
Switch(config-vlan)# name Sales
Switch(config-vlan)# vlan 20
Switch(config-vlan)# name ER
Switch(config-vlan)# exit

! Assign access ports to VLAN 10 (first two ports)
Switch(config)# interface fastEthernet 0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
Switch(config-if)# interface fastEthernet 0/2
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10

! Assign access ports to VLAN 20 (next two ports)
Switch(config)# interface fastEthernet 0/3
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 20
Switch(config-if)# interface fastEthernet 0/4
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 20

! Configure trunk port to router
Switch(config)# interface gigabitEthernet 0/1
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk allowed vlan 10,20
Switch(config-if)# end

Switch# write memory
```
##
## Block 4: Router Configuration

```markdown
## Router Configuration (Subinterfaces)

Enable the physical interface and create subinterfaces for each VLAN.

```cisco
Router> enable
Router# configure terminal

! Activate physical interface
Router(config)# interface gigabitEthernet 0/0
Router(config-if)# no shutdown
Router(config-if)# exit

! Subinterface for VLAN 10 (Sales)
Router(config)# interface gigabitEthernet 0/0.10
Router(config-subif)# encapsulation dot1Q 10
Router(config-subif)# ip address 192.168.10.1 255.255.255.0
Router(config-subif)# no shutdown
Router(config-subif)# exit

! Subinterface for VLAN 20 (ER)
Router(config)# interface gigabitEthernet 0/0.20
Router(config-subif)# encapsulation dot1Q 20
Router(config-subif)# ip address 192.168.1.1 255.255.255.0
Router(config-subif)# no shutdown
Router(config-subif)# end

Router# write memory
```

## Block 5: PC Configuration

```markdown
## PC Configuration

Set static IP addresses on each PC.

| PC          | IP Address      | Subnet Mask     | Default Gateway |
|-------------|-----------------|-----------------|-----------------|
| Sales PC1   | 192.168.10.12   | 255.255.255.0   | 192.168.10.1    |
| Sales PC2   | 192.168.10.13   | 255.255.255.0   | 192.168.10.1    |
| ER PC1      | 192.168.1.2     | 255.255.255.0   | 192.168.1.1     |
| ER PC2      | 192.168.1.3     | 255.255.255.0   | 192.168.1.1     |
