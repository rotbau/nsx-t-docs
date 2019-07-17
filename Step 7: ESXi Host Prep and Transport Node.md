# ESXi Host NSX Prep and Transport Node

## Add ESXi Hosts to NSX Hosts
7.1 NSX Manager UI, Fabric > Nodes > Hosts > click **+Add**
  * **Name:** esxi-01
  * **IP Address:** Host IP
  * **OS:** ESXi
  * **Username:** root
  * **Password:** Password
  * Click Add
  * Confirm Thumbprint - Yes
  * NSX will install VIBs on host. 
    * **Deployment Status:** NSX Install in Progress
    * VIB install will take several minutes.  When complete you will see:
    * **Deployment Status:** NSX installed
    * **Manager Connectivity:** Up
  * Repeat for ALL hosts you want prepped for NSX

## Create Host Uplink Profile
7.2 NSX Manager UI, Fabric > Profiles > Uplink Profiles > click **+ADD**
  * **Name:** host-uplink-profile
  * **Teaming Policy:** Failover
  * **Active Uplinks:** Type in name (uplink-1)
  * **Transport VLAN:** 89 (on my switch the TEP vlan is not native and needs VLAN supplied.  Vlan will vary depending on topology)
  * **MTU:** 1600
  * Click Add

## Create Transport Nodes for ESXi Hosts
7.3 NSX Manager UI, Fabric > Nodes > Transport Nodes > click **+Add**
  * General Tab
    * **Name:** esxi-01-tn
    * **Node:** Select matching host from pull-down
    * **Transport Zones:** Select tz-overlay and move to Selected
  * N-VDS Tab
    * **NVDS Name:** Select nvds1 from the list (example).  You will only see N-VDSs that are part of the TZ's you added above
    * **NIOC Profile:** select nsx-default-nioc-hostswitch-profile
    * **Uplink Profile:** host-uplink-profile (created above)
    * **IP Assignment:** Use IP Pool
    * **IP Pool:** tep-ip-pool
    * **Physical Nics:** vmnic2 (or whatever unused vmnic you have available thats properly configured for TEP network)
    * Add
* Repeat for all Hosts you want to add as transport nodes

7.4 Verify Node(s)
  * NSX Manager UI, Fabric > Nodes > Transport Nodes
    * **Configuration State:** Success
    * **Status:** Up - may take a few moments

7.5 Verify NSX TEP vmk is created on all hosts
  * SSH to ESXi Host(s)
    * `esxi-01> esxcfg-vmknic -l` look for vmk which has Netstack type vxlan (vmk10)
    * `vmkping -S vxlan -I vmk10 -d -s 1550 [VTEP IP]`
    or old command is 
    * `vmkping ++netstack=vxlan {ip of another tep} -d -s 1550` should see reply from other TEPs in the network (either other hosts or edge)
    * `esxcfg-nics -l` to check for MTU of vnics attached to TEP networks.  Needs to be at least 1600
    * Also validate that the vSWITCH or DVS that the NSX EDGE VMs are attached to have MTU of 1600 or greater set.
    * Verify PING from to all EDGE vtep and host vtep interfaces at 1550 bytes