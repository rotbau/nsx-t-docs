#Create NSX objects for PKS Mgmt Plane

## Create Dedicated T1 for PKS Mgmt Plane
1. Create new T1 Router instance for PKS Mgmt Objects (Opsman, PKS Controller, Harbor, etc)
2. Use same process as detailed in Step 7 Logical Switch, T1 router and Route Advertisement
  * **Logical Switch Name:** pks-mgmt-ls (example)
  * **T1 Name:** T1-PKS-MGMT (example)
  * **Configure router advertisement:** enable and NSX connected routes

## Create T1 Router Downlink port
1. Create downlink port and connect to pks-mgmt-ls
  * IP address needs to be at least a /28 for PKS Mgmt plane components (PoC) or /24 in production.  In this design our pks-mgmt network will be routable so we do not need to connect DNAT objects (see documention)

## Verify T1-T0 Connection
T1 should auto connect to T0 router but verify connection by navigating to NSX Manager UI, Networking > Routing > Routers, Click on the T1 router you just connected > Overview Tab > Tier-0 Routers.  You should see your T0-LR router and the 'Disconnect' link.  If you see 'Connect' instead, click that to connect to the T0
  