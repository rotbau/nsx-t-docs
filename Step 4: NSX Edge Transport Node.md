# Create NSX Edge Transport Node and Edge Cluster

Transport nodes are capable of participating in the overlay or VLAN networking.  Any node that contains N-VDS can be a transport node

NSX Edge can belong to one overlay transport zone and multiple VLAN transport zone

Step 3 must be complete before moving forward with this step

## Create TEP IP Pool
1. In NSX Manager UI, Inventory > Groups > IP Pool
2. Click +Add
  * **Name:** tep-ip-pool
  * add IP range, gateway, network address in CIDR (192.168.10.0/24), DNS optional, DNS suffic optional

## Create Transport Zones
1. In NSX Manager UI navigate to Fabric > Transport Zones > Add

### Overlay Transport Zone
1. Used for overlay traffic
  * **Name:** tz-overlay
  * **Host Switch Name:** nvds01
  * **Traffic Type:** Overlay

### VLAN Transport Zone
1. Used for uplink to physical network
  * **Name:** tz-vlan
  * **Host Switch Name:** nvds02
  * **Traffic Type:** VLAN

## Create Edge Uplink Profile
1. Uplink profile used by Edge Nodes
  * **Name:** edge-uplink-profile
  * **Teaming Policy:** Failover
  * **Active Uplinks:** Type in name (uplink-1)
  * **Transport VLAN:** 0 (on my switch the transport vlan is native on the port this pnic is connected to.  Vlan will vary depending on topology)
  * **MTU:** 1600

NSX-T 2.3 Ref:
https://docs.vmware.com/en/VMware-NSX-T-Data-Center/2.3/com.vmware.nsxt.install.doc/GUID-F739DC79-4358-49F4-9C58-812475F33A66.html

## Create Edge Transport Node

1. Log into NSX Manager UI and navigate to Fabric > Nodes > Edges
2. Click actions > Configure as Transport node
3. General Tab
  * **Name:** edge-tn
  * **Transport Zones:** Select both tz-overlay and tz-vlan and move them to Selected
4. N-VDS tab (used to be called Host Switches Tab)
  * Configure Overlay N-VDS
    * Click +Add N-VDS
    * **Edge Switch Name Pull Down:** Select first N-VDS listed.  The N-VDS switches listed were created in the steps above where you created the transport zone.  Select nvds01 - which corresponds to the OVERLAY transport zone in our example
    * **Uplink Profile:** edge-uplink-profile 
    * **IP Assignment:** Use IP Pool
    * **IP Pool:** tep-ip-pool
    * **Virtual Nics:** fp-eth0 (corresponds to MAC of 2nd Nic on Edge VM - vnci1)  **Note:** This nic should be connected to the correct port group you want to use for TEP interfaces
  * Configure VLAN N-VDS
    * **Edge Switch Name Pull Down:** Select nvds02 - which corresponds to the VLAN transport zone in our example
    * **Uplink Profile:** edge-uplink-profile
    * **Virtual Nics:** fp-eth1 (corresponds to MAC of 3rd Nic on Edge VM - vmnic2)  **Note:** This nic should be connected to the correct port group you want to use to uplink to physical network router
4. Save 
5. Validate Functionality
  * NSX Manager UI, Fabric > Nodes > Edges.
  * Verify controller connectivity and Manager Connectivity are 'UP' for all Edges that have been configured.
  * NSX Manager UI, Fabric > Nodes > Transport Node
  * Verify configuration state is 'success'
6. CLI Commands for Edge to verify functionality
  * `nsx-edge> get controllers` verify node is connected to all controllers

## Create Edge Cluster

1. NSX Manager UI, Fabric > Nodes > Edge Clusters
2. Click +Add
  * **Name:** edge-cluster01
  * **Edge Cluster Profile:** Default or create new one if needed
  * **Member Type:** Edge Node
  * **Selected:** Select edge nodes in this installed as desired
  3. Validate Functionality
    * NSX Manager UI, Fabric > Nodes > Edge Clusters, verify you see new edge clusters and all members are present
    * Edge CLI
      * `nsx-edge-1> get vteps`
      * `nsx-edge-1> get host-switches`
      * `nsx-edge-1> get edge cluster status`
      * `nsx-edge-1> get controller sessions`
  4. If you have multiple Edges verify Edge node connectivity between all nodes (TEP to TEP)
    * Edge CLI
      * `nsx-edge-1> get logical-router`
      * `nsx-edge-1> vrf0`
      * `nsx-edge-1>(vrf)> ping IP-ADDRESS-EDGE-2`
