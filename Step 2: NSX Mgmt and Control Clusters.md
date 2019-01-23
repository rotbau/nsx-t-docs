# Create NSX Mgmt and Control Clusters

Once the appliances have all been deployed and powered on you will need to create the management and control clusters.

## Join NSX Controller with NSX Manager
1. SSH to NSX Manager
2. Open separate SSH to NSX Controller
3. Get API thumbprint on NSX Manager.  On NSX Manager

`NSX-Mgr> get certficate api thumbprint`
  * Copy thumbprint to notepad for use in the next command and during edge configuration
4. On each NSX Controller join the management plane

`NSX-Controller> join management-plane (mgr IP) username admin thumbprint (nsx-manager-thumbprint)`
  * provide NSX admin password when prompted
  * look for message 'Node successfully registered and controller restarted'
5. Verify result on NSX Controller

`NSX-Controller> get managers`
  * should see IP of manager with status 'Connected'
6. Verify cluster status on NSX Manager

`NSX-Mgr> get management-cluster status`
  * look for 'Management cluster status: STABLE' and IP address of nodes.
  * Note: may see 'Control cluster status: UNSTABLE' -   This is normal and will change after you initialize the control cluster

  NSX-T 2.3 Ref: https://docs.vmware.com/en/VMware-NSX-T-Data-Center/2.3/com.vmware.nsxt.install.doc/GUID-05434745-7D74-4DA8-A68E-9FE17093DA7B.html#GUID-05434745-7D74-4DA8-A68E-9FE17093DA7B

  ## Initialize Control Cluster to create Master
  1. SSH to NSX Controller and run command

  `NSX-Controller> set control-cluster security-model shared-secret secret (secret passphrase)`

  2. Initialize Cluster

  `NSX-Controller> initialize control-cluster`

  3. Verify controller is master

  `NSX-Controller> get control cluster-status verbose`

  * look for 'is master: true', 'is majority: true', and Zookeeper Server IP: IP, reachable, ok'
  * Note: may take a few minutes for this to update.  Repeat command if needed

  4. Validate on NSX Manager

  `NSX-Mgr> get management-cluster status`

  * should now see 'Control cluster status: STABLE'

  NSX 2.3 Ref:
  https://docs.vmware.com/en/VMware-NSX-T-Data-Center/2.3/com.vmware.nsxt.install.doc/GUID-273F6344-7212-4105-9FBA-A872CD75803F.html#GUID-273F6344-7212-4105-9FBA-A872CD75803F