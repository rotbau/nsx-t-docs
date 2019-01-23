 # NSX Edge Cluster Join Management Plane

Once the NSX Edge VM has been deployed and configured it needs to be registered to the NSX Manager

## Join NSX Edge with NSX management plane
1. SSH to NSX Manager
2. Open separate SSH to NSX Edge
3. Get API thumbprint on NSX Manager or reuse the one you obtained when joining controllers.  On NSX Manager

`NSX-Mgr> get certficate api thumbprint`

4. On NSX Edge join the management plane

`NSX-Edge join management-plane (mgr IP) username admin thumbprint (nsx-manager-thumbprint)`
  * provide NSX admin password when prompted
  * look for message 'Node successfully registered and edge restarted'
5. Verify result on NSX Edge

`NSX-Edge> get managers`
  * should see IP of manager with status 'Connected'
6. Verify cluster status on NSX Manager but going to UI and Fabric > Edges.  You should see Manager Connectivity Up.

 NSX-T 2.3 Ref:
 https://docs.vmware.com/en/VMware-NSX-T-Data-Center/2.3/com.vmware.nsxt.install.doc/GUID-AECC66D0-C968-4EF2-9CAD-7772B0245BF6.html

 https://docs.vmware.com/en/VMware-NSX-T-Data-Center/2.3/com.vmware.nsxt.install.doc/GUID-11BB4CF9-BC1D-4A76-A32A-AD4C98CBF25B.html#GUID-11BB4CF9-BC1D-4A76-A32A-AD4C98CBF25B