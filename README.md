# 🌐 Enterprise Multi-Protocol Routing & Core Infrastructure (OSPF & BGP)

[![Cisco](https://img.shields.io/badge/Cisco-Packet%20Tracer%20v8+-049fd9?style=for-the-badge&logo=cisco&logoColor=white)](https://www.netacad.com/courses/packet-tracer)
[![Routing](https://img.shields.io/badge/Protocols-OSPF%20%7C%20BGP-orange?style=for-the-badge)](https://en.wikipedia.org/wiki/Border_Gateway_Protocol)
[![Network](https://img.shields.io/badge/Architecture-Enterprise%20Multi--Tier-brightgreen?style=for-the-badge)]()
[![Status](https://img.shields.io/badge/Status-Verified%20%26%20Stable-success?style=for-the-badge)]()

An enterprise-grade network simulation implementing multi-protocol routing across autonomous systems. Designed and simulated in **Cisco Packet Tracer**, this project showcases hierarchical **VLSM subnetting**, intra-domain **OSPF** core convergence, inter-domain **BGP (eBGP & iBGP)** peering, **L2/L3 switch** segmentation, and centralized **enterprise servers** (DNS, DHCP, Web).

---

## 📑 Table of Contents

- [Overview](#-overview)
- [Key Features](#-key-features)
- [Architecture & Topology](#-architecture--topology)
- [Repository Structure](#-repository-structure)
- [Protocol Specifications](#-protocol-specifications)
  - [VLSM IP Addressing](#1-vlsm-ip-addressing--subnetting)
  - [OSPF Configuration (IGP)](#2-ospf-interior-gateway-protocol)
  - [BGP Configuration (EGP)](#3-bgp-border-gateway-protocol)
  - [Route Redistribution](#4-mutual-route-redistribution)
  - [Enterprise Server Farm](#5-enterprise-server-services)
- [Verification & Troubleshooting](#-verification--troubleshooting)
- [Getting Started](#-getting-started)
- [Author](#-author)

---

## 📖 Overview

In modern enterprise environments, networks must provide low-latency local traffic delivery while securely exchanging routes with external Autonomous Systems (AS) and service providers. 

This project simulates a complete, progressive enterprise network topology:
1. **Intra-Domain Routing (IGP)**: Employs **OSPF** to achieve sub-second convergence, loop-free paths, and scalable multi-area hierarchy inside the enterprise core.
2. **Inter-Domain Routing (EGP)**: Leverages **BGP** for robust policy routing, multi-homed WAN connectivity, and internet gateway peering across separate Autonomous Systems.
3. **Switching & Segmentation**: Deploys multi-layer switches (L3) with 802.1Q VLAN trunking and SVI (Switch Virtual Interface) inter-VLAN routing to isolate departmental broadcast domains.
4. **Core Infrastructure Services**: Hosts dedicated enterprise servers handling automated host address provisioning (DHCP), domain name resolution (DNS), and intranet web services (HTTP).

---

## ✨ Key Features

- **Hierarchical Subnetting (VLSM)**: Optimized IP allocation minimizing address waste across point-to-point router links (`/30`), server blocks, and client subnets.
- **Dynamic OSPF Core**: Scalable interior routing with optimized Cost metrics, designated router (DR/BDR) election handling, and neighbor adjacency tuning.
- **Multi-AS BGP Peering**: Configured external BGP (eBGP) across AS borders and internal BGP (iBGP) for route reflection within boundary routing groups.
- **Mutual Route Redistribution**: Bidirectional route exchange with route-maps and metric filtering between OSPF and BGP at Autonomous System Boundary Routers (ASBRs).
- **L2/L3 VLAN Aggregation**: Switched network infrastructure with access/trunk port hardening, native VLAN security, and high-speed inter-VLAN routing.
- **Centralized Server Farm**: Integrated DNS, DHCP, and Web services providing end-to-end host verification across both local and cross-AS remote endpoints.

---

## 🏛 Architecture & Topology

```
                  +-----------------------------------+
                  |         BGP WAN / INTERNET        |
                  |         [ Autonomous System ]     |
                  +-----------------+-----------------+
                                    |
                            (eBGP Peering Link)
                                    |
                  +-----------------+-----------------+
                  |          ASBR ROUTER              |
                  |     (OSPF <-> BGP Redistribution) |
                  +-----------------+-----------------+
                                    |
                         (OSPF Area 0 / Core)
                                    |
         +--------------------------+--------------------------+
         |                                                     |
+--------+--------+                                   +--------+--------+
|   DISTRIBUTION  |                                   |  SERVER FARM    |
|   L3 SWITCH     |                                   |  (DNS/DHCP/WEB) |
+--------+--------+                                   +-----------------+
         |
  (802.1Q Trunk)
         |
+--------+--------+
|  ACCESS SWITCH  |
+--------+--------+
         |
   +-----+-----+
   |           |
[VLAN 10]   [VLAN 20]
(Staff)     (Engineering)
```

---

## 📂 Repository Structure

The project represents a multi-stage progression from baseline connectivity to complete multi-protocol integration:

| File | Type | Description |
| :--- | :--- | :--- |
| **`project 1.pkt`** | Cisco Packet Tracer | Initial baseline network topology and fundamental IP addressing setup. |
| **`project 2.pkt`** | Cisco Packet Tracer | Layer 2/3 switch configuration, VLAN segmentation, and trunk links. |
| **`project 3.pkt`** | Cisco Packet Tracer | Inter-VLAN routing deployment (Router-on-a-Stick / SVI) & client tests. |
| **`project 4.pkt`** | Cisco Packet Tracer | Multi-router topology with Single/Multi-Area OSPF dynamic routing. |
| **`project 5.pkt`** | Cisco Packet Tracer | Enterprise server farm integration (DHCP pools, DNS entries, Web server). |
| **`project 6.pkt`** | Cisco Packet Tracer | **Comprehensive Capstone:** Full OSPF core + BGP ASBR peering & redistribution. |
| **`week 1.xlsx`** | Spreadsheet | Comprehensive IP Addressing Table, Subnet Calculations (VLSM), and Port Matrix. |

---

## ⚙ Protocol Specifications

### 1. VLSM IP Addressing & Subnetting
- Point-to-point serial and gigabit inter-router links use efficient `/30` prefixes.
- Departmental subnets are sized via Variable Length Subnet Masking (VLSM) based on anticipated host capacity to preserve IPv4 address space.

### 2. OSPF (Interior Gateway Protocol)
- **Process ID & Router-ID**: Deterministic `router-id` assigned on loopback interfaces.
- **Area Architecture**: Area 0 (Backbone Area) interconnecting all distribution and core layers.
- **Passive Interfaces**: Configured on client and server facing interfaces to eliminate unnecessary Hello packet flooding and enhance security.

```ios
router ospf 1
 router-id 1.1.1.1
 network 10.0.0.0 0.0.0.3 area 0
 network 192.168.10.0 0.0.0.255 area 0
 passive-interface GigabitEthernet0/1
```

### 3. BGP (Border Gateway Protocol)
- Configured on AS Boundary Routers (ASBR).
- Neighbor peering established over WAN links with keepalive/holdtime timer matching.

```ios
router bgp 65001
 bgp router-id 10.255.255.1
 neighbor 203.0.113.2 remote-as 65002
 network 192.168.0.0 mask 255.255.0.0
```

### 4. Mutual Route Redistribution
Redistribution between OSPF and BGP at the border gateway:
```ios
! Injecting BGP routes into OSPF
router ospf 1
 redistribute bgp 65001 subnets metric 20

! Injecting OSPF routes into BGP
router bgp 65001
 redistribute ospf 1
```

### 5. Enterprise Server Services
- **DHCP**: Centralized pool distribution for automated workstation IP leasing.
- **DNS**: Forward lookups for enterprise intranet domains.
- **HTTP/Web Server**: Hosting internal portal to validate application-layer connectivity.

---

## 🔍 Verification & Troubleshooting

To verify routing convergence and network operational health in Cisco Packet Tracer:

### Routing Table Verification
```ios
show ip route
show ip route ospf
show ip route bgp
```

### Protocol Adjacency Verification
```ios
show ip ospf neighbor
show ip bgp summary
show ip bgp
```

### Switching & VLAN Verification
```ios
show vlan brief
show interfaces trunk
show ip interface brief
```

### End-to-End Connectivity
- Multi-subnet ICMP ping: `ping <destination-ip>`
- Route path inspection: `traceroute <destination-ip>`

---

## 🚀 Getting Started

1. **Prerequisites**: Download and install **[Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer)** (Version 8.0 or newer recommended).
2. **Clone the Repository**:
   ```bash
   git clone https://github.com/billiampalad/ospf-bgp.git
   cd ospf-bgp
   ```
3. **Open the Project**:
   - Double-click `project 6.pkt` to launch the final complete topology.
   - Alternatively, inspect `project 1.pkt` through `project 5.pkt` to observe the step-by-step evolution.
   - Refer to `week 1.xlsx` for the complete IP planning and port map.

---

## 👤 Author

- **GitHub**: [@billiampalad](https://github.com/billiampalad)
- **Repository**: [billiampalad/ospf-bgp](https://github.com/billiampalad/ospf-bgp)
