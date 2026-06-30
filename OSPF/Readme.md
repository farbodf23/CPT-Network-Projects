# 🌐 OSPF Dynamic Routing Lab – Cisco Packet Tracer

## Overview

This project demonstrates **dynamic routing** using the **Open Shortest Path First (OSPF)** protocol in Cisco Packet Tracer.

Unlike static routing, where routes must be configured manually on every router, OSPF allows routers to automatically discover neighboring routers, exchange routing information, and calculate the shortest path to every network using Dijkstra's Shortest Path First (SPF) algorithm.

The topology consists of three routers connected in a linear topology, each advertising its directly connected networks through **OSPF Area 0**.

---

## Network Topology

```
LAN1
192.168.10.0/24
      |
      |
     PC1
192.168.10.10
      |
      |
+-------------+
|   Router1   |
+-------------+
 G0/0     G0/1
192.168.10.1
      |
10.0.12.1/30
      |
10.0.12.2/30
 G0/0     G0/1
+-------------+
|   Router2   |
+-------------+
      |
10.0.23.1/30
      |
10.0.23.2/30
 G0/1     G0/0
+-------------+
|   Router3   |
+-------------+
192.168.30.1
      |
      |
192.168.30.10
     PC2

LAN2
192.168.30.0/24
```

---

## IP Addressing Plan

| Device | Interface | IP Address | Subnet Mask |
|---------|-----------|------------|-------------|
| PC1 | Fa0 | 192.168.10.10 | 255.255.255.0 |
| Router1 | G0/0 | 192.168.10.1 | 255.255.255.0 |
| Router1 | G0/1 | 10.0.12.1 | 255.255.255.252 |
| Router2 | G0/0 | 10.0.12.2 | 255.255.255.252 |
| Router2 | G0/1 | 10.0.23.1 | 255.255.255.252 |
| Router3 | G0/1 | 10.0.23.2 | 255.255.255.252 |
| Router3 | G0/0 | 192.168.30.1 | 255.255.255.0 |
| PC2 | Fa0 | 192.168.30.10 | 255.255.255.0 |

---

## Network Overview

| Network | Purpose |
|---------|---------|
| 192.168.10.0/24 | LAN connected to Router1 |
| 10.0.12.0/30 | Point-to-point link between Router1 and Router2 |
| 10.0.23.0/30 | Point-to-point link between Router2 and Router3 |
| 192.168.30.0/24 | LAN connected to Router3 |

---

## OSPF Configuration

All routers participate in **OSPF Process 1** using **Area 0 (Backbone Area)**.

### Router1

```cisco
enable
configure terminal

hostname R1

interface GigabitEthernet0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown

interface GigabitEthernet0/1
 ip address 10.0.12.1 255.255.255.252
 no shutdown

router ospf 1
 network 192.168.10.0 0.0.0.255 area 0
 network 10.0.12.0 0.0.0.3 area 0

end
write
```

---

### Router2

```cisco
enable
configure terminal

hostname R2

interface GigabitEthernet0/0
 ip address 10.0.12.2 255.255.255.252
 no shutdown

interface GigabitEthernet0/1
 ip address 10.0.23.1 255.255.255.252
 no shutdown

router ospf 1
 network 10.0.12.0 0.0.0.3 area 0
 network 10.0.23.0 0.0.0.3 area 0

end
write
```

---

### Router3

```cisco
enable
configure terminal

hostname R3

interface GigabitEthernet0/0
 ip address 192.168.30.1 255.255.255.0
 no shutdown

interface GigabitEthernet0/1
 ip address 10.0.23.2 255.255.255.252
 no shutdown

router ospf 1
 network 192.168.30.0 0.0.0.255 area 0
 network 10.0.23.0 0.0.0.3 area 0

end
write
```

---

## PC Configuration

### PC1

| Setting | Value |
|---------|-------|
| IP Address | 192.168.10.10 |
| Subnet Mask | 255.255.255.0 |
| Default Gateway | 192.168.10.1 |

### PC2

| Setting | Value |
|---------|-------|
| IP Address | 192.168.30.10 |
| Subnet Mask | 255.255.255.0 |
| Default Gateway | 192.168.30.1 |

---

## How OSPF Works in This Lab

After OSPF is enabled:

1. Each router sends **Hello packets** on OSPF-enabled interfaces.
2. Neighboring routers discover one another and establish **adjacencies**.
3. Routers exchange **Link-State Advertisements (LSAs)**.
4. Every router builds an identical **Link-State Database (LSDB)**.
5. Each router independently runs the **Shortest Path First (SPF)** algorithm.
6. The resulting routes are automatically installed into the routing table.

No static routes are configured anywhere in the network.

---

## Verification

### View Neighbor Relationships

```cisco
show ip ospf neighbor
```

Displays neighboring routers and their adjacency state.

---

### View the Routing Table

```cisco
show ip route
```

Routes learned through OSPF appear with the code:

```
O
```

Example:

```
O 192.168.30.0/24
O 10.0.23.0/30
```

---

### View the Link-State Database

```cisco
show ip ospf database
```

Displays the Link-State Advertisements (LSAs) exchanged between routers.

---

### View OSPF Interface Information

```cisco
show ip ospf interface
```

Shows OSPF settings, interface costs, timers, and operational status.

---

### Test Connectivity

From **PC1**:

```
ping 192.168.30.10
```

The packet traverses:

```
PC1
 ↓
Router1
 ↓
Router2
 ↓
Router3
 ↓
PC2
```

without any manually configured static routes.

---

## Concepts Demonstrated

- Dynamic Routing
- Open Shortest Path First (OSPF)
- OSPF Process Configuration
- OSPF Area 0
- Neighbor Discovery
- Hello Packets
- Link-State Advertisements (LSAs)
- Link-State Database (LSDB)
- Dijkstra's Shortest Path First (SPF) Algorithm
- Automatic Route Learning
- Point-to-Point Networks (/30)
- Routing Table Verification
- End-to-End Connectivity Testing

---

## Learning Outcome

This project demonstrates how routers can automatically exchange routing information and dynamically adapt to topology changes using OSPF. Compared to static routing, OSPF significantly improves scalability, reduces administrative overhead, and enables routers to compute the optimal path to every destination without manually configuring individual routes
