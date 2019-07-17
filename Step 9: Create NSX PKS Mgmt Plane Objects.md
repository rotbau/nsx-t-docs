# Create NSX objects for PKS Mgmt Plane

If you are planning on using NSX with VMware PKS then this section will cover the NSX objects that need to be created to

## Network Planning
9.1 Subnets Required for PKS
  * TEP IP Pool
    * **Use:** ESXi and Edge Nodes TEP interfaces
    * **IP Pool:** 192.168.89.10-192.168.89.x (this was created back in step 4.1)
    * **Network Routable:** YES
  * T0 Router Uplink IP
    * **Use:** IP for uplink of T0 to physical netowrk router and for BGP routing traffic
    * **IP:** 192.168.79.3 (used in step 6.4)
    * **Network Routable:** YES
  * PKS MGMT Subnet
    * **Use:** Opsman, BOSH, PKS, Harbor vms configured on this network.
    * **IP Subnet:** 172.31.0.0/24
    * **Network Routable:** YES (note: can use NAT for this but then need to configure SNAT and DNAT rules)
  * PKS Service Subnet
    * **Use:** Needs to be there but not used.  Holdover from old design but you do need to create them.
    * **IP Subnet:** 172.31.2.0/24
    * **Network Routable:** YES (but not used)
  * Nodes IP Block
    * **Use:** Kubernetes Master and Nodes. Each new cluster peels a block from this subnet (/24 for now)
    * **IP Subnet:** 172.15.0.0/16 **see warnings
    * **Network Routable:** NO
  * PODs IP Block
    * **Use:** PODs (containers) network
    * **IP Subnet:** 172.16.0.0/16 **see warnings
    * **Network Routable:** NO
  * VIP IP Pool
    * **Use:** VIPs used for exposed services are taken from this pool
    * **IP Subnet:** 10.40.14.32/27
    Range 10.40.14.32-10.40.14.62
    * **Network Routable:** NO

  **Warnings:** ** Do not use any of the RFC 1918 blocks that Kubernetes or Harbor uses internally:

  * Docker Daemon on Kubernetes worker nodes uses subnets in the following CIDR.  **Do not use IPs from following CIDR range:**
    * 172.17.0.0/16
  * Kubernetes clusters subnet for Kubernetes service.  **Do not use IPs from following CIDR range:**
    * 10.100.200.0/24
  * PKS w/ Harbor - Harbor uses the following CIDR for internal docker bridges.  **Do not use IPs from following CIDR range:**
    * 172.18.0.0/16
    * 172.19.0.0/16
    * 172.20.0.0/16
    * 172.21.0.0/16
    * 172.22.0.0/16

## Create Switches for PKS MGMT and PKS SERVICE

9.2 Create PKS Managerment Logical Switch
  * Opsman, BOSH, PKS vms connect to this switch
  * NSX Manager UI, Networking > Switching > Switches > click **+Add**
    * **Name:** pks-mgmt-ls
    * **Transport Zone:** tz-overlay
    * **Uplink:** Teaming: Use Default
    * **Replication Mode:** Hierarchiacal Two-Tier replication
    * **VLAN:** Leave blank
    * No Changes on switching profiles tab
    * Click Add

9.3 Create PKS Service Logical Switch
  * Not Used but needs to bee created
  * NSX Manager UI, Networking > Switching > Switches > click **+Add**
    * **Name:** pks-service-ls
    * **Transport Zone:** tz-overlay
    * **Uplink:** Teaming: Use Default
    * **Replication Mode:** Hierarchiacal Two-Tier replication
    * **VLAN:** Leave blank
    * No Changes on switching profiles tab
    * Click Add

## Create Tier-1 Logical Router for PKS MGMT and PKS Service

9.4 Create PKS Management T1 Router Instance
  * NSX Manager UI, Networking > Routing > click **+Add** > Tier-1-Router
  * **Name:** T1-PKS-MGMT
  * **Tier-0-Router:** T0-LR
  * **Edge Cluster:** LEAVE BLANK**
  * Click Add
  * Note: router port to T0 automatically created and plumbed
  * ** Do not attach 1 T1 to Edge Cluster unless you specifically need services like NAT, DFW, LB or other stateful service.  Doing so will cause the T1 to not be distributed.

  9.5 Create PKS Servie T1 Router Instance
  * NSX Manager UI, Networking > Routing > click **+Add** > Tier-1-Router
  * **Name:** T1-PKS-SERVICE
  * **Tier-0-Router:** T0-LR
  * **Edge Cluster:** LEAVE BLANK**
  * Click Add
  * Note: router port to T0 automatically created and plumbed
  * ** Do not attach 1 T1 to Edge Cluster unless you specifically need services like NAT, DFW, LB or other stateful service.  Doing so will cause the T1 to not be distributed.

## Create Tier-1 Router Ports and Advertisement

9.6 Configure T1 Ports and Route Advertisement for PKS Management
  * NSX Manager UI, Networking > Routing
  * Locate T1-PKS-MGMT router, click on it opening the router properties.
  * Configuration Pull-down > Router Ports > click **+Add**
    * **Name:** pks-mgmt-ls-routerport
    * **Type:** Downlink
    * **URPF Mode:** Leave at default
    * **Logical Switch:** pks-mgmt-ls
    * **Logical Switch Port:** Attach to new switch port.
    * **Switchport name:** give name (pks-mgmt-ls-routerport) or leave blank for auto-generated port name
    * **IP Address/mask:** 172.31.0.254/24
    * Click Add
  
  9.7 Configure T1 Route Advertisement for PKS Management
    * * Locate T1-PKS-MGMT router, click on it opening the router properties.
  * Routing Pull-down > Route Advertisement > click **Edit**
    * **Status:** Enable
      * Advertise All NSX Connected Routes
      * Advertise All NAT Routes (only if using NAT for MGMT network)
    * Save

9.6 Configure T1 Ports for PKS Service
  * NSX Manager UI, Networking > Routing
  * Locate T1-PKS-SERVICE router, click on it opening the router properties.
  * Configuration Pull-down > Router Ports > click **+Add**
    * **Name:** pks-service-ls-routerport
    * **Type:** Downlink
    * **URPF Mode:** Leave at default
    * **Logical Switch:** pks-service-ls
    * **Logical Switch Port:** Attach to new switch port.
    * **Switchport name:** give name (pks-service-ls-routerport) or leave blank for auto-generated port name
    * **IP Address/mask:** 172.31.2.254/24
    * Click Add

9.8 Configure T1 Route Advertisement for PKS Service
    * * Locate T1-PKS-SERVICE router, click on it opening the router properties.
  * Routing Pull-down > Route Advertisement > click **Edit**
    * **Status:** Enable
      * Advertise All NSX Connected Routes
      * Advertise All NAT Routes (only if using NAT for MGMT network)
    * Save

## Create IP Blocks for PKS Components
9.9 Create Kubernetes Nodes IP Block
  * NSX Manager UI, Networking > IPAM > click **+Add**
    * **Name:** pks-nodes-ip-block
    * **CIDR:** 172.15.0.0/16

9.10 Create PODS IP Block
  * NSX Manager UI, Networking > IPAM > click **+Add**
    * **Name:** pks-pods-ip-block
    * **CIDR:** 172.16.0.0/16

9.11 Create VIP IP Pool
  * NSX Manager UI, Inventory > Groups > click **+Add**
    * **Name:** pks-vip-ip-pool
    * +Add
    * **IP Ranges:** 10.40.14.34-10.20.14.62
    * **CIDR:** 10.40.14.32/27
    * Click Add

## Configure Routing for VIP IP Range
9.12 Configure Routing on T0 if using BGP
  * NSX Manager UI, Networking > Routing
  * Locate TO router, click on it opening the router properties.
  * Routing Pull-down > Static Routes > click **+Add**
    * **Network:** 10.40.14.32/27 (VIP network)
    * **Next Hops:** +Add
      * **Next Hop:** NULL
      * Click Add

9.13 Configure/Reconfigure Route Redistribution with BGP
  * T0 > Routing Pull-down > Route Redistribution
  * You should already have a **bgp-route-redistribution** created from step 6.7
  * Select and click **Edit** on bgp-route-redistribution
  * Click **T0 Static** in addition to the already selected NSX Static.  Both should be checked
  * Save
  * Verify in your Physical Switch that you are peering with that the 10.40.14.32/27 network is now listed.

9.14 Static Route (optional only use if not using BGP)
  * On your Physical Router / Switch for your environment create a new static route to the 10.40.14.32/27 network with 192.168.79.3 (edge uplink configured in step 6.4)

  ## Create NSX API Certificate for use by PKS

  During install NSX generated a self-signed certificate that was based on hostname.  PKS expects a certificated based on the FQDN so we will need to generate and replace the NSX certificate.

9.14 Create new nsx.crt and nsx.key for API access
  * SSH to CLI-VM (linux jump host with PKS and Kubectl tools install)
  * Create a directory to hold script and objects
    * `mkdir ~/nsx-cert`
    * `cd ~/nsx-cert`
  * Create nsx-cert.cnf file
    * `vi nsx-cert.cnf`
    * Copy / Past the below code into the nsx-cert.cnf file.  Replace nsxmgr.corp.local and IP address with your NSX Manager FQDN and IP
    ````
    [ req ]
    default_bits = 2048
    distinguished_name = req_distinguished_name
    req_extensions = req_ext
    prompt = no
    [ req_distinguished_name ]
    countryName = US
    stateOrProvinceName = California
    ocalityName = Palo-Alto
    organizationName = NSX
    commonName = nsxmgr.corp.local
    [ req_ext ]
    subjectAltName=DNS:nsxmgr.corp.local,IP:192.168.110.42
    ````
  * Export variables on your CLI-VM (jumphost) commandline replacing the FQDN and IP with your information
    * `export NSX_MANAGER_IP_ADDRESS=192.168.110.42`
    * `export NSX_MANAGER_COMMONNAME=nsxmgr.corp.local` 
  * Paste the following command to the CLI-VM command line and execute it
    ````
    openssl req -newkey rsa:2048 -x509 -nodes \
    -keyout nsx.key -new -out nsx.crt -subj /CN=$NSX_MANAGER_COMMONNAME \
    -reqexts SAN -extensions SAN -config <(cat ./nsx-cert.cnf \
    <(printf "[SAN]\nsubjectAltName=DNS:$NSX_MANAGER_COMMONNAME IP:$NSX_MANAGER_IP_ADDRESS")) -sha256 -days 365
    ````

  * The above file will create 2 files, **nsx.crt** and **nsx.key**

9.16 Cat the contents of the **nsx.crt** and **nsx.key** files to validate contents

9.17 Add the new API Certificate to NSX Manager

  * NSX Manager UI > System > Trust > Certificates > click **Import** > Import Certificate (Make sure you do not choose CA Certificate)
    * **Name:**  NSX_API_CERT
    * **Certificate Kay:** Paste contents of the nsx.crt file including the `'------ BEGIN CERTIFICATE` and `END CERTIFICATE -----'`
    * **Private Key:** Paste contents of the nsx.key file including the `'------ BEGIN CERTIFICATE` and `END CERTIFICATE -----'`
    * **Password:** NSX Manager Password
    * Click Import

9.18 Register new Certifcate for API Access

* IN NSX Manager click on ID of the NSX_API_CERT created in step 9.15.
  * Copy the ID to clipboard
* On CLI-VM command line run the following command pasting the ID copied above in place of the [certificateID]
  * `export CERTIFICATE_ID=[certificateID]`
* Excecute the following command from CLI-VM command line
  ````
  curl --insecure -u admin:'VMware1!' \
  -X POST \
  "https://$NSX_MANAGER_IP_ADDRESS/api/v1/node/services/http?action=apply_certificate&certificate_id=$CERTIFICATE_ID"
  ````
* Log Out of NSX Manager and Log back in

Creation of NSX PKS Objects is now complete.  Your NSX-T installation is now ready to support VMware PKS


