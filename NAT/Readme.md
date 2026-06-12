# 🌐 Static NAT Lab – Cisco Packet Tracer

## Overview

This lab demonstrates **Static Network Address Translation (Static NAT)** using Cisco Packet Tracer.  
An internal HTTP server with a private IP is exposed to an external network through a fixed public IP mapping on a dedicated NAT router.

| | Address |
|---|---|
| **Private (inside) IP** | `192.168.20.10` |
| **Public (outside) IP** | `203.0.113.10` |
| **NAT Router** | Router B |

---

## Network Topology
[ PC LAN ]──────[ Router A ]──────[ Router B ]──────[ HTTP Server ]

192.168.10.0/24     │       203.0.113.0/24      │    192.168.20.0/24

G0/0/0 ◄──────────────────► G0/0

G0/0/1                      G0/1

(outside)                 (inside)

### IP Address Table

| Device / Interface | IP Address | Subnet Mask | Connected To |
|---|---|---|---|
| PC | 192.168.10.x | 255.255.255.0 | Router A – G0/0/0 |
| Router A – G0/0/0 | 192.168.10.1 | 255.255.255.0 | PC LAN |
| Router A – G0/0/1 | 203.0.113.1 | 255.255.255.0 | Router B – G0/0 |
| Router B – G0/0 *(outside)* | 203.0.113.2 | 255.255.255.0 | Router A – G0/0/1 |
| Router B – G0/0 *(secondary)* | 203.0.113.10 | 255.255.255.0 | *(virtual – NAT public IP)* |
| Router B – G0/1 *(inside)* | 192.168.20.1 | 255.255.255.0 | HTTP Server |
| HTTP Server | 192.168.20.10 | 255.255.255.0 | Router B – G0/1 |

> **Note:** No dynamic routing is used. All routes are directly connected or static.

---

## How Static NAT Works Here

When the PC sends a packet to `203.0.113.10`:
1. Router A resolves the destination via ARP on the `203.0.113.0/24` link.
2. Router B replies (because it owns `203.0.113.10` as a secondary address).
3. The packet arrives at Router B, which translates the destination to `192.168.20.10`.
4. The packet is forwarded to the HTTP server on the inside interface.

### Why a Secondary IP Was Needed

Without it, Router B only owns `203.0.113.2` and will **not** answer ARP for `.10` — causing *"Destination Unreachable"* errors. Adding `203.0.113.10` as a secondary address on the outside interface fixes this.

> **Alternative:** Use a `/30` transit link and a host route (`/32`) for the public IP to avoid the need for a secondary address. The secondary IP approach is simpler for lab environments.

---

## Router Configurations

### Router A – Pure Routing (No NAT)

```cisco
hostname RouterA
!
interface GigabitEthernet0/0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown
!
interface GigabitEthernet0/0/1
 ip address 203.0.113.1 255.255.255.0
 no shutdown
!
! Optional – only needed if PC requires internet access
ip route 0.0.0.0 0.0.0.0 203.0.113.2
end
```

### Router B – Static NAT

```cisco
hostname RouterB
!
interface GigabitEthernet0/0
 description *** Outside – Link to Router A ***
 ip address 203.0.113.2 255.255.255.0
 ip address 203.0.113.10 255.255.255.0 secondary
 ip nat outside
 no shutdown
!
interface GigabitEthernet0/1
 description *** Inside – Link to HTTP Server ***
 ip address 192.168.20.1 255.255.255.0
 ip nat inside
 no shutdown
!
ip nat inside source static 192.168.20.10 203.0.113.10
end
```

### HTTP Server

- **IP:** `192.168.20.10` / **Mask:** `255.255.255.0` / **Gateway:** `192.168.20.1`
- Enable the HTTP service and add a simple `index.html` page.

### PC (Client)

- **IP:** `192.168.10.2` (any `.2`–`.254`) / **Mask:** `255.255.255.0` / **Gateway:** `192.168.10.1`

---

## Verification

**1. Check Router A's ARP table** — you should see Router B's MAC mapped to `203.0.113.10`:
```cisco
RouterA# show arp
```

**2. Check NAT translations on Router B:**
```cisco
RouterB# show ip nat translations
```
Expected output:
Pro  Inside global    Inside local     Outside local  Outside global

---  203.0.113.10     192.168.20.10    ---            ---

**3. Test from the PC:**
C:> ping 203.0.113.10
Also open the PC's web browser and navigate to `http://203.0.113.10`.

**4. Debug NAT in real time (if needed):**
```cisco
RouterB# debug ip nat
```
Then re-run the ping to watch the translation happen live.

---

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| `Destination Unreachable` from PC | Router A can't resolve `203.0.113.10` via ARP | Add `203.0.113.10` as secondary IP on Router B's outside interface |
| Nothing in `show ip nat translations` | NAT not triggered; interfaces not marked | Verify `ip nat inside` / `ip nat outside` and the static NAT command |
| Ping works but HTTP doesn't | HTTP service disabled or firewall active | Enable HTTP on server; disable PC firewall in CPT |
| Server can't reach PC | No return route on Router B | Add: `ip route 192.168.10.0 255.255.255.0 203.0.113.1` |

---
