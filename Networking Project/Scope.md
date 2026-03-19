## Objective

The goal is to simulate a 5-site Proof of Concept (PoC) for a functioning enterprise, modeling a complete and secure network infrastructure. This involves creating a Hierarchical Hub-and-Spoke Overlay within a Proxmox-based GNS3 environment to demonstrate high availability, disaster recovery, and multi-vendor integration.

---

## 1. ISP Backbone (The Underlay)

This node represents the Public Internet and acts as the transit point for all site-to-site connectivity.

- **Model:** Cisco IOU L3 (Image #13: `i86bi-linux-l3-jk9s-15.0.1.bin`)
    
- **Role:** The Public Internet / Service Provider.
    
- **Hardware Config:** 4 Adapters (16 Ethernet Ports).
    
- **Rationale:** Provides high port density to connect all corporate locations to a single "ISP" node, significantly reducing RAM overhead compared to multiple router instances.


This plan uses the **10.x.y.z** hierarchy:

- **X (2nd Octet):** The Site ID (e.g., 10 for NY, 20 for SJ).
    
- **Y (3rd Octet):** The VLAN/Service ID (e.g., 10 for Data, 99 for Management).
    
- **Z (4th Octet):** The Host ID (e.g., .1 for Gateway, .50+ for users).

|**Connection**|**ISP Side IP**|**Site Side IP**|**Subnet**|
|---|---|---|---|
|ISP to NY-EDGE-1|203.0.113.1|203.0.113.2|203.0.113.0/30|
|ISP to NY-EDGE-2|203.0.113.5|203.0.113.6|203.0.113.4/30|
|ISP to SJ-EDGE|203.0.113.9|203.0.113.10|203.0.113.8/30|
|ISP to CHI-BRANCH|203.0.113.13|203.0.113.14|203.0.113.12/30|
|ISP to ATL-BRANCH|203.0.113.17|203.0.113.18|203.0.113.16/30|

The internal network follows a strict `10.[Site].[VLAN].0/24` format.

### VLAN/Service Identification

- **.10:** Management (Network Devices)
    
- **.20:** Corporate Data (Users/PCs)
    
- **.30:** Voice (VoIP Phones)
    
- **.50:** Server Farm (Logistics DB, Web)
    
- **.254:** Transit (Inter-router links)


---

## 2. Primary Hub: New York Data Center (NY-DC)

The main corporate headquarters and the primary exit point for East Coast internet traffic.

- **Role:** Primary Corporate Hub and Data Center.
    
- **Edge Routers (2x):** Cisco Catalyst 8000v (C8K).
    
    - **Rationale:** Successor to the CSR 1000v. Running C8K demonstrates familiarity with modern Cisco virtual platforms and SD-WAN-capable hardware.
        
- **Core Firewall (1x):** FortiGate-VM64 (FortiOS 7.4+).
    
    - **Rationale:** Fortinet is an industry leader. This handles "North-South" security, providing deep packet inspection for traffic entering the internal network.
        
- **Core Switches (2x):** Cisco IOU L3 (Image #13).
    
    - **Rationale:** Configured as a collapsed core to manage high-speed internal routing and VLAN management for the server farm.
        
- **Servers (2x):** Ubuntu Docker Containers.
    
    - **Roles:** One for the Web Server (Logistics Portal) and one for Management (Ansible automation and Syslog).
        


### NY-DC Hub (Site ID: 10)

- **Management:** 10.10.10.0/24
    
- **User LAN:** 10.10.20.0/24
    
- **Server Farm:** 10.10.50.0/24
    
- **Internal Core Link:** 10.10.254.0/30

---

## 3. Secondary Hub: San Jose Data Center (SJ-DC)

The secondary regional hub providing geographic redundancy and disaster recovery.

- **Role:** Disaster Recovery (DR) Hub and West Coast Internet Exit.
    
- **Edge Router (1x):** Cisco Catalyst 8000v (C8K).
    
    - **Rationale:** Maintains feature parity with the primary hub for seamless failover.
        
- **Core Firewall (1x):** FortiGate-VM64.
    
- **Core Switch (1x):** Cisco IOU L3.
    
- **Server (1x):** Ubuntu Docker Container.
    
    - **Role:** Database Mirror (Synchronizes data with NY-DC).
        


### SJ-DC Hub (Site ID: 20)

- **Management:** 10.20.10.0/24
    
- **User LAN:** 10.20.20.0/24
    
- **Database Mirror:** 10.20.50.0/24
    
- **Internal Core Link:** 10.20.254.0/30
---

## 4. Tier 1 Branch: Chicago Logistics Center (CHI)

A high-volume location requiring constant uptime through redundant connections.

- **Role:** Tier 1 Branch (Redundant / High Volume).
    
- **Branch Edge Routers (2x):** MikroTik CHR (Cloud Hosted Router).
    
    - **Rationale:** Extremely lightweight (256MB RAM) while supporting full BGP and IPsec features. Dual routers allow for the simulation of Dual-ISP failover.
        
- **Distribution Switch (1x):** Cisco IOU L3.
    
- **End Device (1x):** GNS3 Network-Toolbox (Docker).
    
    - **Role:** Power-user workstation for testing network throughput, latency, and pathing.

### Chicago Branch (Site ID: 30)

- **Management:** 10.30.10.0/24
    
- **Logistics User LAN:** 10.30.20.0/24
    
- **VoIP Subnet:** 10.30.30.0/24

---

## 5. Tier 2 Branch: Atlanta Regional Office (ATL)

A standard regional office representing a typical small-to-medium branch deployment.

- **Role:** Tier 2 Branch (Standard / Economy).
    
- **Branch Edge Router (1x):** MikroTik CHR.
    
- **Access Switch (1x):** Cisco IOU L2 (Image #1: `i86bi-linux-l2-adventerprise-15.1b.bin`).
    
    - **Rationale:** Demonstrates the ability to implement a standard Layer 2 access environment connected to a Layer 3 gateway.
        
- **End Device (1x):** VPCS (Virtual PC Simulator).
    
    - **Role:** Basic employee workstation for verifying connectivity via Ping and DNS.


### Atlanta Branch (Site ID: 40)

- **Management:** 10.40.10.0/24
    
- **Standard User LAN:** 10.40.20.0/24

---


|**Location**|**Connectivity Strategy**|**Redundancy Level**|
|---|---|---|
|NY-DC|Primary Hub|High (Dual Edge Routers + HA Firewall)|
|SJ-DC|Disaster Recovery Hub|Medium (Single Edge + Single Firewall)|
|Chicago|Dual-Homed (Tunnels to NY and SJ)|High (Dual Branch Routers)|
|Atlanta|Single-Homed (Primary to NY, Backup to SJ)|Standard (Single Branch Router)|
|ISP|Full Mesh Simulation|Essential (Core Backbone)|

## Implementation Phases

1. **Phase 1:** Deploy ISP Backbone and establish Public IP reachability for all sites.
    
2. **Phase 2:** Configure NY-DC and SJ-DC Edge Routers with BGP peering to the ISP.
    
3. **Phase 3:** Implement the Overlay (DMVPN or Site-to-Site IPsec) to secure internal traffic.
    
4. **Phase 4:** Deploy internal switching and firewall policies at the Hubs.
    
5. **Phase 5:** Test Failover scenarios by simulating a total outage of the NY-DC.
