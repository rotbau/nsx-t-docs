# NSX Component Install
1. Deploy NSX Manager
  * Deploy NSX unified appliance OVA
  * Connect Nic to mgmt vlan
  * Configure password and IP parameters as prompted
2. Deploy NSX controller
  * Deploy NSX-T controller appliance OVA.  Single appliance for lab, 3 appliances for real world
  * Connect Nic to mgmt vlan
  * Configure password and IP parameters as prompted
  * Leave Management Cluster section blank
3. Deploy NSX Edge
  * Deploy NSX-T Edge appliance OVA
  * Connect NICs to appropriate VLANs
    * Nic 1 = MGMT Portgroup/vlan
    * Nic 2 = TEP vlan (1600 mtu minimum)
    * Nic 3 = vlan for edge uplink to physical router.  Connect to Portgroup/vlan where T0 uplink will be located
    * Nic 4 = Not used so connect to any Portgroup/vlan

  * For PKS installations Edge node needs to be type Large
  * Configure password and IP parameters as prompted
  4. Validate all appliance installs are successful and management IPs are all pingable.

  For more help reference:
  NSX-T 2.3 Manager
  https://docs.vmware.com/en/VMware-NSX-T-Data-Center/2.3/com.vmware.nsxt.install.doc/GUID-FA0ABBBD-34D8-4DA9-882D-085E7E0D269E.html
 
  NSX-T 2.3 Controller
  https://docs.vmware.com/en/VMware-NSX-T-Data-Center/2.3/com.vmware.nsxt.install.doc/GUID-24428FD4-EC8F-4063-9CF9-D8136740963A.html

  NSX-T 2.3 Edge Node
  https://docs.vmware.com/en/VMware-NSX-T-Data-Center/2.3/com.vmware.nsxt.install.doc/GUID-AECC66D0-C968-4EF2-9CAD-7772B0245BF6.html
