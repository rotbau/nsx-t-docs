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
  * Add
  * Verify Node functionality; 'Configuration State: Success'; 'Status: Up'  Note: This may take a few moments
  * Repeat for all ESXi hosts you want prepped