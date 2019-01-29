# Create TEP IP Pool and Transport Zones

## Create TEP IP Pool
4.1 NSX Manager UI, Inventory > Groups > IP Pool > click **+Add**
  * **Name:** tep-ip-pool
  * Click **+Add** under Subnets
    * **IP range** (192.168.89.10-192.168.89.20 example
    * **Gateway** 192.168.89.254
    * **CIDR** (192.168.89.0/24)
    * **DNS** (optional)
    * **DNS suffix** (optional)
    * Click Add button

## Create Transport Zones
4.2 NSX Manager UI, Fabric > Transport Zones > click **+Add**
  * VLAN Transport Zone (used for uplink to physical network)
    * **Name:** tz-vlan
    * **Host Switch Name:** nvds01 (example)
    * **Traffic Type:** VLAN
    * Click Add button

* Overlay Transport Zone (used for overlay traffic)
    * **Name:** tz-overlay
    * **Host Switch Name:** nvds02 (example)
    * **Traffic Type:** Overlay
    * Click Add button