## Project Overview
This repository contains the architecture, topology design, and implementation phases for a 5-site production-grade Enterprise Network Proof of Concept (PoC). Engineered within a virtualized GNS3 environment hosted on a bare-metal Proxmox VE hypervisor, this project simulates a secure, resilient, and highly available corporate infrastructure. 

The architecture models a **Hierarchical Hub-and-Spoke Topology** that integrates multi-vendor virtual appliances (Cisco, Fortinet, and MikroTik) to demonstrate high availability (HA), geographic disaster recovery (DR) synchronization, and edge security enforcement.

---

## Architectural Topology Matrix

The simulation models a geographically distributed enterprise consisting of primary hubs, secondary hubs, and varying tiers of regional branch offices:

| Location | Functional Role | Edge Security & Routing Stack | Redundancy Level |
| :--- | :--- | :--- | :--- |
| **NY-DC** | Primary Corporate Hub / East Coast Data Center | 2x Cisco Catalyst 8000v, 1x FortiGate-VM64, 2x Cisco IOU L3 | **High** (Dual Edge Routers + HA Firewall Appliance) |
| **SJ-DC** | Disaster Recovery (DR) Hub / West Coast Exit | 1x Cisco Catalyst 8000v, 1x FortiGate-VM64, 1x Cisco IOU L3 | **Medium** (Single Edge Router + Single Firewall) |
| **CHI-BRANCH** | Tier 1 Logistics Center (High-Volume Site) | 2x MikroTik Cloud Hosted Routers (CHR), 1x Cisco IOU L3 | **High** (Dual Edge Routers / Dual-ISP Failover) |
| **ATL-BRANCH** | Tier 2 Regional Branch (Economy Site) | 1x MikroTik Cloud Hosted Router (CHR), 1x Cisco IOU L2 | **Standard** (Single Edge Router / Single Gateway) |
| **ISP Core** | Public Transit Backbone (Underlay) | 1x Cisco IOU L3 (High Port Density Instance) | **Critical Backbone** (Full Mesh Underlay Simulation) |

---

## IP Addressing & Subnet Hierarchy

The internal network topology adheres to a deterministic, structured variable-length subnet masking (VLSM) schema based on the `10.[Site_ID].[VLAN_ID].0/24` format.

* **Second Octet (`X`):** Corresponds to the global Site Identification (e.g., `10` = New York Data Center, `20` = San Jose Data Center, `30` = Chicago, `40` = Atlanta).
* **Third Octet (`Y`):** Corresponds to the logical VLAN / Service Segment Isolation.

### Logical Service Segmentation (VLANs)
* **`.10` (Management):** Out-of-Band management access for network infrastructure infrastructure.
* **`.20` (Corporate Data):** End-user corporate computing devices.
* **`.30` (Voice):** Dedicated Quality of Service (QoS) subnets for VoIP telephony.
* **`.50` (Server Farm / Storage):** Internal database mirrors, logistics web portals, and microservices.
* **`.254` (Transit):** Point-to-point transit networks reserved for inter-router communication blocks.

### WAN Underlay Interconnect Matrix
Public WAN connections are mapped across `/30` point-to-point networks to conserve public IPv4 space and mimic ISP-demarcation standards:

| Interconnect Segment | ISP Boundary IP | Edge Customer Edge (CE) IP | Network Subnet |
| :--- | :--- | :--- | :--- |
| **ISP ──> NY-EDGE-1** | 203.0.113.1 | 203.0.113.2 | 203.0.113.0/30 |
| **ISP ──> NY-EDGE-2** | 203.0.113.5 | 203.0.113.6 | 203.0.113.4/30 |
| **ISP ──> SJ-EDGE** | 203.0.113.9 | 203.0.113.10 | 203.0.113.8/30 |
| **ISP ──> CHI-BRANCH** | 203.0.113.13 | 203.0.113.14 | 203.0.113.12/30 |
| **ISP ──> ATL-BRANCH** | 203.0.113.17 | 203.0.113.18 | 203.0.113.16/30 |

---

## Multi-Vendor Design Rationale

* **Cisco Catalyst 8000v (C8K):** Deployed at the primary and secondary enterprise hubs. Utilizing the C8K validates capability with modern virtualized architectures, IOS-XE syntax, and enterprise-grade SD-WAN/routing features.
* **FortiGate-VM64 (FortiOS 7.4+):** Positioned between the Edge and the Core switches to govern North-South data boundaries. FortiGate instances handle deep packet inspection (DPI), stateful security filtering, and zone access controls.
* **MikroTik Cloud Hosted Router (CHR):** Implemented at the branch office tiers. These instances provide full BGP and IPsec protocol capabilities while optimizing compute resources (requiring only 256MB RAM per node).
* **Cisco IOU L3/L2 (IOS on Unix):** Utilized for internal enterprise core, distribution, and access switching layers to ensure high-performance wirespeed switching and collapsed core deployment efficiency.

---

## Implementation Blueprint

```text
┌──────────────────────────┐     ┌──────────────────────────┐     ┌──────────────────────────┐
│  Phase 1: ISP Underlay   │ ──> │  Phase 2: Hub Edge BGP   │ ──> │   Phase 3: Overlay VPN   │
└──────────────────────────┘     └──────────────────────────┘     └──────────────────────────┘
                                                                               │
┌──────────────────────────┐     ┌──────────────────────────┐                  │
│ Phase 5: Failover Verification │ <── │ Phase 4: LAN & Security Core │ <────────────┘
└──────────────────────────┘     └──────────────────────────┘
