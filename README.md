# ISP-Simulation-GNS3

This project is a complete simulation of an ISP (Internet Service Provider) infrastructure using **GNS3**, **Cisco IOU**, and **Docker-based containers**. The scenario includes multiple interconnected **Autonomous Systems (AS)**, implementing routing protocols, NAT techniques, VPNs, DHCP services, security testing, and BGP route manipulation techniques. It was developed in a Linux (Ubuntu) environment.

---

## Project Structure

### Network Design


- **Data Center (DC)** — Located in AS65002 with public service IPs (e.g., `40.0.0.0/24`).
- **Branch Offices** — Internal LANs connected to customer routers (C1, C2) using NAT and VPNs.
- **Loopback0 Interfaces** — Used as stable BGP router IDs and for testing reachability across ASes.
- **AS12353** (Main ISP) — Core backbone, BGP/OSPF integration, route reflectors.
- **AS174**, **AS65001**, **AS65002** — External ISPs simulating inter-provider connectivity.
- **Routers**: R1 to R7 (Cisco IOU-based), with full mesh iBGP or route reflectors where appropriate.
- **Clients and Services**: Internal networks simulated with Docker containers (netutils), web servers, DNS, and DHCP clients.
- **Firewalls**: Implemented using **OPNsense** for the perimeter and security filtering.

---

## Addressing	

- Applied **VLSM** (Variable Length Subnet Masking) for efficient subnetting.
- Ensured unique Loopback IPs for BGP Router IDs and testing.
- Used real-world hierarchical addressing to simulate regional and global routes.

---

## NAT / PAT

- **Static NAT**, **Dynamic NAT**, and **PAT (NAT overload)** were configured on border routers (e.g., C2R1).
- Inside and outside interfaces were correctly identified.
- Route-maps with ACLs were used to control NAT translation scope.

---

## OPNsense Firewall

- Deployed OPNsense to act as an edge firewall.
- Configured filtering rules, interfaces, and routing to simulate real perimeter protection.
- Allowed/denied traffic based on interface zones and NAT.

---

## Routing Protocols

###**BGP (Border Gateway Protocol)**

- **eBGP** used between external AS connections (e.g., R1 ↔ R6, R1 ↔ R7).
- **iBGP** used within AS12353 using **route reflectors** to avoid full mesh.
- **Prefix Filtering**: Only global prefixes and Loopbacks (e.g., `1.1.1.1` to `8.8.8.8`) advertised
- Configured **route-maps**, **prefix-lists**, and **AS-Path Prepending** to control routing policies.
- **Default routes injected** into OSPF instead of full BGP redistribution.

###**OSPF (Open Shortest Path First)**

- OSPF is used as the **IGP** inside AS12353 and other ASes.
- Proper area design was considered (e.g., backbone area 0).
- Loopbacks and LAN interfaces were advertised to ensure reachability.

---

### lo0 192.168.1.0/24 
- **LAN subnet**: `192.168.1.0/24`
- **Router**: C1R1
- **NAT/PAT**: Enabled on C1R1 with NAT overload for internal hosts accessing external networks.
- **Firewall**: OPNsense connected between LAN and ISP router (WAN/LAN interface split).
- **DHCP Server**: Assigns dynamic IPs to internal clients via Netutils Docker or Linux container.
- **Security**:
  - ACLs applied to control NAT translation.
  - Simulated **DHCP starvation attack** with Kali Linux to test DoS resilience.

### lo0 192.168.2.0/24
- **LAN subnet**: `192.168.2.0/24`
- **Router**: C2R1
- **VRF Implementation**: C2 is placed under a `VRF C2` to simulate **MPLS/VPN customer isolation**.
- **BGP VPNv4**: Used to advertise `192.168.2.0/24` into the ISP backbone via `Route Distinguisher 12353:30`.
- **NAT**: PAT with route-map matching internal source addresses only.

### Loopback Interfaces
- **Purpose**:
  - Loopback0 interfaces (e.g., `192.168.1.10`, `192.168.2.10`) simulate internal servers.
  - Also used for ping and routing validation across AS boundaries.
- **Announced in BGP**:
  - Only Loopback0 interfaces are allowed to be propagated by BGP speakers as per the project constraints.

---

## MPLS & VRF (Basic)

- Used **VRFs (Virtual Routing and Forwarding)** for service separation in the ISP.
- BGP VPNv4 address families enabled on R1 and R5 to simulate **MPLS VPN** environments.
- Verified route distinguishers and import/export between VRFs.

---

## Security Testing

###**BGP Prefix Hijacking Simulation**

- AS65001 announced a route (40.0.0.0/24) falsely via R7.
- Manipulated attributes to redirect traffic via malicious AS.
- Demonstrated detection and removal of the hijacked prefix.

###**DHCP Starvation Attack**

- DHCP server deployed in internal network.
- Used Docker-based Kali container to simulate **starvation attacks**.
- Showcased detection and mitigation with firewall/DHCP filtering.

---

## VPNs

- Configured **IPsec VPNs** between branch offices.
- Tunnels terminated at OPNsense/firewalls to simulate secure external connections.
- Verified encryption and routing over secure paths.

---

## Services and Infrastructure

- **DHCP Server**: Distributed dynamic IPs to clients inside LANs.
- **DNS/Web Servers**: Used to simulate real internet services.
- **Capture Interfaces**: Used `.pcap` captures for validation and testing (excluded from repo).

---

## Repository Contents

ISP-Simulation-GNS3/
├── project-files/ # GNS3 topology, configs, and captures
├── README.md # This file
├── .gitignore # Excludes large .pcap files
├── Relatorio/ # Report with screenshots and analysis
└── Captures/ # Optional captures (excluded from Git)


---

## Tests Performed

- ✅ Route preference testing (local-pref, weight)
- ✅ Path manipulation (AS-Path Prepending)
- ✅ NAT translation tests (ICMP, HTTP)
- ✅ VPN reachability validation
- ✅ BGP hijacking scenario (detection + mitigation)
- ✅ DHCP starvation attempt
- ✅ OSPF and BGP convergence

---

## Developed & Tested On

- **OS**: Ubuntu Linux
- **Tools**: GNS3, Wireshark, Docker, Cisco IOU, OPNsense, Kali Linux (security testing)
- **Topology**: Based on academic project design (see `projeto-atualizado.pdf`)

---
