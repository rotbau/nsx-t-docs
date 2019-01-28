# Create NSX Edge Transport Node and Cluster

Transport nodes are capable of participating in the overlay or VLAN networking.  Any node that contains N-VDS can be a transport node

NSX Edge can belong to one overlay transport zone and multiple VLAN transport zone

Steps 3 and 4 must be complete before moving forward with this step

## Create Edge Uplink Profile
5.1 NSX Manager UI, Fabric > Profiles > Uplink Profiles > click **+Add**
  * **Name:** edge-uplink-profile
  * **Teaming Policy:** Failover
  * **Active Uplinks:** Type in name (uplink-1)
  * **Transport VLAN:** 0 (because VM Nics are mapped to port groups with VLANs already configured this can be left at 0)
  * **MTU:** 1600

## Create Edge Transport Node
5.2 NSX Manager UI, Fabric > Nodes > Edges
  * Click actions > Configure as Transport node
    * General Tab
    * **Name:** edge-tn
    * **Transport Zones:** Select both tz-overlay and tz-vlan and move them to Selected
    * N-VDS tab (previously Host Switches Tab)
      * Create OVERLAY N-VDS
        * **Edge Switch Name Pull Down:**  Select nvds02 - which corresponds to the OVERLAY transport zone in our example
        * **Uplink Profile:** edge-uplink-profile 
        * **IP Assignment:** Use IP Pool
        * **IP Pool:** tep-ip-pool
        * **Virtual Nics:** fp-eth0 (corresponds to MAC of 2nd Nic on Edge VM - vnci1)  **Note:** This nic should be connected to the correct port group you want to use for TEP interfaces
        * Click **+ Add N-VDS** towards top of page to add another N-VDS for VLAN
      * Configure VLAN N-VDS
        * **Edge Switch Name Pull Down:** Select nvds01 - which corresponds to the VLAN transport zone in our example
        * **Uplink Profile:** edge-uplink-profile
        * **Virtual Nics:** select fp-eth1 (corresponds to MAC of 3rd Nic on Edge VM - vmnic2)  
        * **Note:** This nic should be connected to the correct port group you want to use to uplink to physical network router
    * Click Add

5.3 Validate Functionality
  * NSX Manager UI, Fabric > Nodes > Edges.
    * **Controller connectivity:** UP
    * **Manager Connectivity:** UP
    * Verify for all Edges that have been configured.
  * NSX Manager UI, Fabric > Nodes > Transport Node
    * **Configuration State:** Success
    * **Status:** UP
  * CLI Commands for Edge to verify functionality
    * `nsx-edge> get controllers` 
      * Verify node is connected to all controllers
      * **Status:** connected
      * **Session State:** up

## Create Edge Cluster

5.4 NSX Manager UI, Fabric > Nodes > Edge Clusters > Click **+Add**
  * **Name:** edge-cluster01
  * **Edge Cluster Profile:** Default or create new one if needed
  * **Member Type:** Edge Node
  * **Selected:** Select edge node(s) and click > to move to selected

5.5 Validate Functionality
  * NSX Manager UI, Fabric > Nodes > Edge Clusters, verify you see new edge clusters and all members are present
    * Edge CLI
      * `nsx-edge-1> get vteps`
      * `nsx-edge-1> get host-switches`
      * `nsx-edge-1> get edge cluster status`
      * `nsx-edge-1> get controller sessions`
  * If you have multiple Edges verify Edge node connectivity between all nodes (TEP to TEP)
    * Edge CLI
      * `nsx-edge-1> get logical-router`
        * verify VTEP IP is present
      * `get logical-routers`
      * `nsx-edge-1> vrf 0`
      * `nsx-edge-1>(vrf)> ping IP-ADDRESS-EDGE-2`
