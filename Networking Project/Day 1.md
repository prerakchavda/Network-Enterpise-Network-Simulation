## Phase 1: Base ISP & Edge Connectivity (Day 1)
Objective

The goal of Day 1 was to establish the core routing environment and simulate external internet connectivity (ISP) to ensure the Edge routers can reach public-facing services.
Task 1: ISP Core Initialization (IOU)

The backbone of the simulation starts with the IOU (IOS on Unix) instance. To transform the device into a functional gateway, the following global and interface-level configurations were applied:

    Global Routing: Enabled ip routing to allow the device to process Layer 3 traffic.

    Terminal Configuration: Optimized the VTY lines and console for administrative access.

    Point-to-Point Links: Configured interfaces with a /30 subnet mask (255.255.255.252).

        Design Choice: Using a /30 ensures only two usable IP addresses (one for the ISP and one for the Router), which conserves address space and mimics real-world WAN link standards.

    Documentation: Applied interface descriptions to maintain clear labeling of network segments.

(Networking Project/Screenshots/Editing Interfaces in IOU.png)
Task 2: Simulating Public Services

To validate the routing logic without an actual internet breakout, a Loopback Interface was created to simulate Google’s Public DNS (8.8.8.8).

    This acts as our "North Star" for connectivity testing throughout the project.

    By making this reachable, we confirm that our internal routing tables are correctly propagating external routes.

![Loopback Config](networking-project/screenshots/Loopback%20Ip%20change.png)Task 3: Edge Router Integration (NY-EDGE-1)

With the ISP core ready, I moved to the first site gateway: NY-EDGE-1.

    Interface Alignment: Manually configured the WAN-facing interface to match the subnet parameters defined in the IOU ISP core.

    Link Activation: Executed the no shutdown command on all relevant ports. This transitioned the physical and data-link layers to an "Up/Up" status, finalizing the physical handshake between the Edge and the ISP.

![Edge Setup](networking-project/screenshots/Setting%20up%20edge%20routers.png)Task 4: Connectivity Verification (The Ping Test)

The final milestone for Day 1 was verifying end-to-end reachability.

    Test: A standard ICMP ping was issued from NY-EDGE-1 to the simulated Google DNS address (8.8.8.8).

    Result: The test was successful, confirming that the Edge router can traverse the ISP link to reach external resources.

![Ping Test](networking-project/screenshots/Ping%20test%20on%208.8.8.8%20from%20edge%20router.png)
