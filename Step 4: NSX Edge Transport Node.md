# Create NSX Edge Transport Node

Transport nodes are capable of participating in the overlay or VLAN networking.  Any node that contains N-VDS can be a transport node

NSX Edge can belong to one overlay transport zone and multiple VLAN transport zone

Step 3 must be complete before moving forward with this step

## Create TEP IP Pool
1. In NSX Manager UI navigate to Inventory > Groups > IP Pool
2. Click +Add
  * add IP range, gateway, network address in CIDR (192.168.10.0/24), DNS optional, DNS suffic optional

## Create Transport Zones
1. In NSX Manager UI navigate to Fabric > Transport Zones > Add

### Overlay Transport Zone
1. Used for overlay traffic
2. Name: tz-overlay
3. Host Switch Name: nvds01
4. Traffic Type: Overlay

### VLAN Transport Zone
1. Used for uplink to physical network
2. Name: tz-vlan
3. Host Switch Name: nvds02
4. Traffic Type: VLAN

### Create Edge Uplink Profile
1. Name: edge-uplink-profile
2. Teaming Policy: Failover
3. Active Uplinks: Type in name (uplink-1)
4. Transport VLAN: 0 (on my switch the transport vlan is native on the port this pnic is connected to.  Vlan will vary depending on topology)
5. MTU: 1600

NSX-T Ref:
https://docs.pivotal.io/runtimes/pks/1-2/nsxt-deploy.html#create-nsx-cluster