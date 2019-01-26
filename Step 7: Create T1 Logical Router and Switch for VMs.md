# Create Tier 1 Logical Router/Switch for VMs

This will cover creating a Tier 1 logical router and switch to provide logical networking for standard VMs.  The T1 Router uplink connects to a T0 for ingress/egress router and downlinks to one or more logical switch for VM workloads.

## Create T1 Router Instance
1. NSX Manager UI, Networking > Routing, +Add Tier-1-Router
  * **Name:** T1-LR-VM
  * **Tier-0-Router:** T0-LR
  * **Edge Cluster:** edge-cluster01 (pull-down)
  * **Failover Mode:** Non-preemptive
  * **Edge Cluster Memebers:** edge-tn
  * Note: router port to T0 automatically created and plumbed

## Create VM overlay logical switch
1. NSX Manager UI, Networking > Switching > Switches, +Add
  * **Name:** app-ls (example)
  * **Transport Zone:** tz-overlay
  * **Uplink:** Teaming: Use Default
  * **Replication Mode:** Hierarchiacal Two-Tier replication
  * **VLAN:** Leave blank
  * No Changes on switching profiles tab
  * Add

## Connect T1 downlink port
1. NSX Manager UI, Networking > Routing
2. Locate T1 router created above and click on it opening the router properties.
3. Configuration -> Router Ports
  * **Name:** appls-downlink-port
  * **Type:** Downlink
  * **URPF Mode:** Leave at default or not
  * **Logical Switch:** app-ls (example)
  * **Logical Switch Port:** Attach to new switch port.
    * **Switchport name:** give name or leave blank for auto-generated port name
  * **IP Address/mask:** x.x.x.x/x This address becomes the default gateway for VMs connected to the app-ls logical switch.  It also determnieds the network for the VMs connected to the app-ls logical switch
  * ADD

## Configure Route Advertisement for T1 router
1. NSX Manager UI, Networking > Routing
2. Locate T1 router created above and click on it opening the router properties.
3. Routing -> Route Advertisement
  * Click Edit
  * **Status:** Enabled
  * **Advertise All NSX Connected Routes:** Yes (note this will advertise the logical networks connected to this T1 to the T0 and north if BGP is configured on the T0)
  
  Note: you can also set static routes if needed to other networks.

## Verify T1-T0 Connection
T1 should auto connect to T0 router but verify connection by navigating to NSX Manager UI, Networking > Routing > Routers, Click on the T1 router you just connected > Overview Tab > Tier-0 Routers.  You should see your T0-LR router and the 'Disconnect' link.  If you see 'Connect' instead, click that to connect to the T0
