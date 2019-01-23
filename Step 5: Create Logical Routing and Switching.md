# Create Logical Routing and Switching

The T0 Router provides ingress/egress to the Physical Network from Overlay Networks (T1 Routers).  The T0 and T1 logical routers are connected via a logical switch.  We will create the base logical routing / switching construct here.

## Create VLAN logical switch
1. NSX Manager UI, Networking > Switching > Switches, +Add
  * **Name:** uplink-ls01
  * **Transport Zone:** tz-vlan
  * **Uplink:** Teaming: Use Default
  * **VLAN:** 0 (vlan used here is native on my switch hence 0, you may have other vlan that needs to be entered here)
  * No Changes on switching profiles tab
  * Save
2. Verify Admin Status is 'UP' and Conf State is 'Success' - May take awhile
## Create T0 Router Instance
1. NSX Manager UI, Networking > Routing, +Add Tier-0-Router
  * **Name:** T0-LR (or LR-PKS if you want)
  * **Edge Cluster:** edge-cluster01 (pull-down)
  * **High Availability:** Active-Standy **Failover Mode** Non-preemptive
## Create T0 Router Port
1. NSX Manager UI, Networking > Routing
2. Select T0 you created (LR-T0) - Click on Name so detail pane opens up
3. Click Configuration > Router Ports, +Add
  * **Name:** edge-uplink1
  * **Type:** Uplink
  * **MTU:** 1500 (example)
  * **Transport Node:** edge-tn1
  * **URPF Mode:** left to Strict (need to clarify)
  * **Logical Switch:** Select VLAN logical switch (uplink-ls01)
  * **IP Address:** 192.168.x.x/24 (note this address is used to peer with upstream physical network and needs to be on same VLAN used in VLAN logical switch and correspond to Edge NIC mapping for fp-eth1 vmnic2)
4. Save
## Configure Static or Dynamic Routing
### Static Route
1. NSX Manager UI, Networking > Routing > Routers and select (click) T0 router created in previous step
2. Select Routing > Static Routes, +Add
### BGP Routing
1. NSX Manager UI, Networking > Routing > Routers and select (click) T0 router created in previous step
2. Select Routing > BGP
3. Add Neighbor (Physical Router/L3 Switch)
  * Neighbor Tab
  * **Neighbor Address:** Physical Router If
  * **Remote AS:** AS configured on Physical 64997
  * **Timers:** As required by network
  * **Password:** leave blank or enter
  * Local Address Tab
  * **All Uplinks:** I deselected this box
  * **Type:** Uplink
  * **Selected:** Move desired available uplink to selected box
  * Note: it is probably OK to leave all all uplinks
  * Save
4. Enable BGP Configuration (hit Edit button on same page)
  * **Status:** Enabled
  * **Graceful Restart:** Enabled (I have a single edge)
  * **Local AS:** 64998 (NSX is own AS in my case)
  Note: need to configure your physical switch for BGP as well to match.  Also need to distribute default route from physical via BGP or else you will still need a static route
### BGP Route Redistribution from T0
1. NSX Manager UI, Networking > Routing > Routers and select (click) T0 router
2. Select Routing > Route Redistribution, +Add
  * **Name:** bgp-route-redistribution
  * **Sources:** NSX Static
  * Add
3. Enable Route Redistribution (hit Edit button on same page)
  * **Status:** Enabled
### Verify BGP Functionality
1. SSH to nsx-edge
2. `nsx-edge> get logical-routers` look for type SERVICE_ROUTER_TIER0.  Take note of VRF#
3. `nsx-edge> vrf 2` (2 is VRF for my T0)
4. `nsx-edge(tier0_sr)> get bgp neighbor`
  * Look for BGP State: Established UP
  * Local host: IP and port should be populated
  * Remote host: IP and port should be populated
5. `nsx-edge(tier0_sr)> get route bgp`
  * should see any routes your physical swtiches are sending.  In my case that the 0s route `b 0.0.0.0/0 [20/0] via 192.168.x.254`
6. Verify you can ping the T0 uplink IP