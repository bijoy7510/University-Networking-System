# Southeast University Network Design and Implementation

A complete enterprise campus network design for **Southeast University (SEU)**, built and simulated in **Cisco Packet Tracer**. The project connects four academic departments to a central server farm and delivers secure public web access via Static NAT.

## 📌 Overview

- **Organization:** Southeast University (SEU)
- **Goal:** Connect academic departments (CSE, EEE, BBA, Mathematics), provide centralized server access, and expose web services securely to the internet.
- **Approach:** Multi-protocol routing (RIPv2 + OSPFv2 Area 0) with mutual redistribution, combined with Static NAT at the perimeter gateway.

## 🖧 Network Requirements

| Item | Details |
|---|---|
| Departments / Networks | 8 internal LANs + 7 WAN links + 1 public WAN (16 subnets total) |
| Devices | 8 × Cisco 1941/2901 Routers, Cisco 2960 Switches, Client PCs |
| Servers | DNS Server `192.168.10.226`, Web Server `192.168.10.227` |
| Routing Protocols | RIPv2, OSPFv2 (Process 1, Area 0) with mutual redistribution |
| NAT | Static NAT on Router 8 — `192.168.10.227` → `203.0.113.10` |

## 🗺️ Topology

Routers 1–8 are connected sequentially via point-to-point `/30` links. The Server Farm is attached to Router 6, and the external public network (`203.0.113.0/24`) with PC24 is attached to Router 8.

## 🌐 IP Addressing Summary

| Department / Segment | Network | Mask | CIDR |
|---|---|---|---|
| CSE Department | 192.168.10.0 | 255.255.255.128 | /25 |
| Academic Segment | 192.168.10.128 | 255.255.255.192 | /26 |
| EEE Department | 192.168.11.0 | 255.255.255.224 | /27 |
| BBA Department | 192.168.10.192 | 255.255.255.224 | /27 |
| Math Department | 192.168.11.64 | 255.255.255.224 | /27 |
| Server Farm | 192.168.10.224 | 255.255.255.240 | /28 |
| Admin / Lab Segments | 192.168.10.240 / .248 | 255.255.255.248 | /29 |
| WAN Links (×7) | 10.0.0.0/30 – 10.0.0.24/30 | 255.255.255.252 | /30 |
| Public WAN | 203.0.113.0 | 255.255.255.0 | /24 |

## ⚙️ Key Configurations

**RIPv2 (all routers)**
```
router rip
 version 2
 no auto-summary
 redistribute ospf 1 metric 2
```

**OSPFv2 (example: Router 8)**
```
router ospf 1
 router-id 8.8.8.8
 network 10.0.0.24 0.0.0.3 area 0
 network 192.168.11.64 0.0.0.31 area 0
 network 203.0.113.0 0.0.0.255 area 0
 redistribute rip subnets
exit
```

**Static NAT (Router 8)**
```
interface GigabitEthernet0/0
 ip nat inside
exit
interface GigabitEthernet0/1
 ip nat inside
exit
interface GigabitEthernet0/2
 ip nat outside
exit
ip nat inside source static 192.168.10.227 203.0.113.10
```

**DNS Records**
| Name | Type | Destination |
|---|---|---|
| university.com | A | 192.168.10.227 |
| ums | A | 192.168.10.227 |

## ✅ Verification

- OSPF adjacency between Router 7 and Router 8 reached `FULL` state.
- Internal PCs successfully resolved `http://university.com` → *"Welcome to Southeast University"*.
- External client (PC24) successfully accessed the web server via `http://203.0.113.10`.
- `show ip nat translations` confirmed active Static NAT mapping.

## 🛠️ Troubleshooting Highlights

- **Router 7–8 Adjacency Issue:** Mismatched IP on Router 7's Gig0/1 (`10.0.0.225`) corrected to `10.0.0.25/30`, restoring OSPF neighbor state to FULL.
- **NAT Outside Subnet Route:** External PC faced web timeouts until the `203.0.113.0/24` network was advertised in OSPF on Router 8, completing end-to-end connectivity.

## 📂 Repository Contents

- `University Networking System.pkt` — Cisco Packet Tracer project file
- `README.md` — Project documentation
- `LICENSE` — Apache 2.0 License

## 🧰 Tools Used

- Cisco Packet Tracer
- RIPv2 & OSPFv2 Routing Protocols
- Static NAT
- VLSM & FLSM Subnetting

## 📄 License

This project is licensed under the [Apache License 2.0](LICENSE).
