# DHCP IP Assignment Lab – Cisco Packet Tracer

## Project Overview

This project demonstrates a **dynamic IP address assignment** using a dedicated DHCP server in a small enterprise network. The topology includes:

- A **multilayer switch** acting as the core router (inter‑VLAN routing not required – the switch simply routes between subnets).
- A **DHCP server** that leases IP addresses to client devices.
- A **DNS server** for name resolution.
- An **HTTP server** hosting a simple web page.
- Several **client PCs** that obtain their IP configuration automatically via DHCP.

The router (multilayer switch) and all servers use **static IP addresses**, while all client PCs receive their IPs from the DHCP server.

---

## IP Addressing Plan

| Device                | IP Address (Static)      | Role / Notes                                      |
|-----------------------|--------------------------|---------------------------------------------------|
| Multilayer Switch     | 192.168.1.1 / 24         | Default gateway for the LAN                      |
| DHCP Server           | 192.168.1.2 / 24         | Provides IPs to clients (scope: .10 – .255)      |
| DNS Server            | 192.168.1.3 / 24         | Resolves domain names (e.g., `http://example.com`)|
| HTTP Server           | 192.168.1.5 / 24         | Serves web content                               |
| Client PCs (DHCP)     | 192.168.1.10 – .254 / 24 | Assigned dynamically by DHCP server              |

> **Note:** The DHCP server excludes addresses `.1`–`.9` (static devices) and only leases addresses from `.10` to `.254`.

---

## Topology Diagram (Text Representation)
