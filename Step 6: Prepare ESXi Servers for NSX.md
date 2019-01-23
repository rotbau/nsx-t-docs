# Prepare ESXi Servers for NSX

## Add ESXi Hosts
1. NSX Manager UI, Fabric > Nodes > Hosts, +Add
  * **Name:** esxi-01
  * **IP Address:** Host IP
  * **OS:** ESXi (or other supported OS)
  * **Username:** root
  * **Password:** Password
  * Save
  * Confirm Thumbprint - Yes
  * NSX will install VIBs on host.  After a few minutes you should see 'Deployment Status: NSX installed' and Manager 'Connectivity: Up'
2. Repeat for other hosts you want prepped for NSX
## Create Host Uplink Profile
1. **Name:** host-uplink-profile
2. **Teaming Policy:** Failover
3. **Active Uplinks:** Type in name (uplink-1)
4. **Transport VLAN:** 89 (on my switch the TEP vlan is not native and needs VLAN supplied.  Vlan will vary depending on topology)
5. **MTU:** 1600
## Create Transport Nodes for ESXi Hosts
1. NSX Manager UI, Fabric > Nodes > Transport Nodes, +Add
2. General Tab
  * **Name:** esxi-01-tn
  * **Node:** Select matching host from pull-down
  * **Transport Zones:** Select tz-overlay and move to Selected
3. N-VDS Tab
  * **NVDS Name:** Select nvds1 from the list (example).  You will only see N-VDSs that are part of the TZ's you added above
  * **NIOC Profile:** select nsx-default-nioc-hostswitch-profile
  * **Uplink Profile:** host-uplink-profile (created above)
  * **IP Assignment:** Use IP Pool
  * **IP Pool:** tep-ip-pool
  * **Physical Nics:** vmnic2 (or whatever unused vmnic you have available thats properly configured for TEP network)
  * Add
  * Verify Node functionality; 'Configuration State: Success'; 'Status: Up'  Note: This may take a few moments
  * Repeat for all ESXi hosts you want prepped
4. Verify NSX TEP vmk is created on all hosts
  * SSH to ESXi Host(s)
  * `esxi-01> esxcfg-vmknic -l` look for vmk which has Netstack type vxlan (vmk10)
  *  `vmkping ++netstack=vxlan {ip of another tep} -d -s 1500` should see reply from other TEPs in the network (either other hosts or edge)
  * `esxcfg-nics -l` to check for MTU of vnics attached to TEP networks