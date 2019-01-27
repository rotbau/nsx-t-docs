# Create NSX Mgmt and Control Clusters

Once the appliances have all been deployed and powered on you will need to create the management and control clusters.

## Join NSX Controller with NSX Manager
2.1 Getp API thumbprint of NSX Manager
  * SSH to NSX Manager VM
    * `NSX-Mgr> get certficate api thumbprint`
    * Copy thumbprint to clipboard and/or text pad

2.2 Join Managment Plane on each NSX Controller
  * Open Separate SSH to NSX Controller VM
    * `NSX-Controller> join management-plane (mgr IP) username admin thumbprint (nsx-manager-thumbprint)`
    * Provide NSX admin password when prompted
    * Look for message **'Node successfull registered and controller restarted'**

2.3 Verify Controller Connectivity to Management Plane
  * On NSX Controller VM
    * `NSX-Controller> get managers`
    * Verify IP of manager with status **'Connected'**
  * On NSX Manager VM
    * `NSX-Mgr> get management-cluster status`
    * Verify  **'Management cluster status: STABLE'** and IP address of nodes.
    * **Note:** You may see 'Control cluster status: UNSTABLE' -   This is normal and will change after you initialize the control cluster

  NSX-T 2.3 Ref: https://docs.vmware.com/en/VMware-NSX-T-Data-Center/2.3/com.vmware.nsxt.install.doc/GUID-05434745-7D74-4DA8-A68E-9FE17093DA7B.html#GUID-05434745-7D74-4DA8-A68E-9FE17093DA7B

  ## Initialize Control Cluster to create Master

  2.4 Set shared secret and initialize cluster
  * SSH to NSX Controller VM
    * Set shared-secret 
      * `NSX-Controller> set control-cluster security-model shared-secret secret (secret passphrase)`
      * Look for **'Security secret successfully set on the node'**

    * Initialize Cluster
     * `NSX-Controller> initialize control-cluster`
     * Look for **'Control cluster initialization successful'**

2.5 Verify control cluster functionality
  * Validate on NSX Controller VM
    * SSH to NSX Controller VM
      * `NSX-Controller> get control cluster-status verbose`
      * Verify
      * **'is master: true'**
      * **'is majority: true'**
      * **Zookeeper Server IP:[IP], reachable, ok'**
      * Note: may take a few minutes for this to update.  Repeat command if needed

  * Validate on NSX Manager VM
    * SSH to NSX Manager VM
      * `NSX-Mgr> get management-cluster status`
      * Verify **'Control cluster status: STABLE'**


  NSX 2.3 Ref:
  https://docs.vmware.com/en/VMware-NSX-T-Data-Center/2.3/com.vmware.nsxt.install.doc/GUID-273F6344-7212-4105-9FBA-A872CD75803F.html#GUID-273F6344-7212-4105-9FBA-A872CD75803F
