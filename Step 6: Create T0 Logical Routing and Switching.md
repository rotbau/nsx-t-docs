# Create Tier 0 Logical Routing and Switching

The T0 Router provides ingress/egress to the Physical Network from Overlay Networks (T1 Routers).  The T0 and T1 logical routers are connected via a logical switch.  We will create the base logical routing / switching construct here.

## Create VLAN logical switch
6.1 NSX Manager UI, Networking > Switching > Switches > click **+Add**
  * **Name:** uplink-vlan-ls
  * **Transport Zone:** tz-vlan
  * **Uplink:** Teaming: Use Default
  * **VLAN:** 0 (vlan used here is native on my switch hence 0, you may have other vlan that needs to be entered here)
  * No Changes on switching profiles tab
  * Click Add

6.2 Verify Status
  * **Admin Status:** UP
  * **Conf State:** Success - May take awhile

## Create T0 Router Instance
6.3 NSX Manager UI, Networking > Routing > Routers > click **+Add** > Tier-0-Router
  * **Name:** T0-LR (or LR-PKS if you want)
  * **Edge Cluster:** edge-cluster01 (pull-down)
  * **High Availability:** Active-Standy 
  * **Failover Mode:** Non-preemptive
  * Click Add
## Create T0 Uplink Router Port
6.4 NSX Manager UI, Networking > Routing
  * Select T0 you created (T0-LR) - Click on Name so detail pane opens up
  * Configuration Pull-down > Router Ports > click **+Add**
    * **Name:** edge-uplink1
    * **Type:** Uplink
    * **MTU:** 1500 (example)
    * **Transport Node:** edge-tn1
    * **URPF Mode:** left to Strict (need to clarify)
    * **Logical Switch:** Select VLAN logical switch (uplink-vlan-ls)
    * **Logical Switch Port:** Give name or leave blank for auto name
    * **IP Address:** 192.168.79.3/24 (note this address is used to peer with upstream physical network and needs to be on same VLAN used in VLAN logical switch and correspond to Edge NIC mapping for fp-eth1 vmnic2)
    * Click Add
## Configure Static or Dynamic Routing
6.5 Static Route (optional depending on BGP config)
  * NSX Manager UI, Networking > Routing > Routers and select (click) T0 router created in previous step
  * Routing Pull-down > Static Routes > click **+Add**
  * Add appropriate IP for physical router on same network as IP configured in step 6.4

6.6 BGP Routing
  * NSX Manager UI, Networking > Routing > Routers and select (click) T0 router created in previous step
  * Routing Pull-down > BGP
    * Click **+Add** under Neighbors
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
        * Click Add
    * Enable BGP Configuration (Click Edit button on same page)
      * **Status:** Enabled
      * **Graceful Restart:** Enabled (I have a single edge)
      * **Local AS:** 64998 (NSX is own AS in my case)
      * Note: need to configure your physical switch for BGP as well to match.  Also need to distribute default route from physical via BGP or else you will still need a static route

6.7 BGP Route Redistribution from T0
  * NSX Manager UI, Networking > Routing > Routers and select (click) T0 router
  * Routing Pull-down > Route Redistribution > click **+Add**
    * **Name:** bgp-route-redistribution
    * **Sources:** NSX Static
    * **NOTE in 2.4.1 For Sources I had to Select T1 Connected (CSP and Downlink) to get route redistribution working.**
    * Validate on NSX Edge VM ``tier0_sr>get bgp neighbor x.x.x.x advertised-routes``
    * Click Add
  * Enable Route Redistribution (hit Edit button on same page)
    * **Status:** Enabled

6.8 Verify BGP Functionality
  * SSH to NSX Edge VM
    * `nsx-edge> get logical-routers` 
      * Look for type SERVICE_ROUTER_TIER0.  Take note of VRF#
    * `nsx-edge> vrf 1` (1 is VRF for my T0)
    * `nsx-edge(tier0_sr)> get bgp neighbor`
      * **BGP State:** Established UP
      * **Local host:** IP and port should be populated
      * **Remote host:** IP and port should be populated
    * `nsx-edge(tier0_sr)> get route bgp`
      * should see any routes your physical swtiches are sending.  In my case that the 0s route `b 0.0.0.0/0 [20/0] via 192.168.x.254`
    * Verify you can ping the T0 uplink IP