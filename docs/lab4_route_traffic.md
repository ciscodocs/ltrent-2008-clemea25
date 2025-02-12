# Lab 4 - Using Data Policy for Routing traffic from Stockholm to Sydney via Singapore firewall in different VRF

## Introduction

In this lab exercise, you will analyze the traffic flow between a user in the **<font color="Green">APAC-Sydney-Branch (site-20)</font>** and a user in the **<font color="Green">EMEA-Stockholm-Branch (site-10)</font>**, with the traffic routed through a **firewall** **<font color="Green">(Singapore-FW)</font>** hosted at the 
**APAC-Singapore-Branch (site-20)** in a different **<font color="#9AAFCB">VRF-2</font>**. The **Singapore-FW**, accessible via **APAC-Singapore-Branch** WAN-Edge router in **<font color="#9AAFCB">VRF-2</font>**, plays a critical role in inspecting and securing the traffic as it traverses the network. This exercise will focus on understanding the configuration and verification of service chaining, 
centralized policies, and the interactions between network elements to ensure the intended traffic flow through the designated security device.

Here is a breakdown of the key components involved in the network path:

- **Source:** The traffic originates from a **Stockholm-User** in the **EMEA-Stockholm-Branch (site-10)**.
- **Destination:** The intended recipient is a **Sydney-User** in the **APAC-Sydney-Branch (site-20)**.
- **Firewall:** All traffic passes through a **firewall (Singapore-FW)**, which is hosted locally at the **APAC-Singapore-Branch (site-20)** in **<font color="green">VRF-2</font>**.
- **WAN Edge Router:** The **Singapore-Branch** WAN-Edge router, facilitates the traffic's reachability to the firewall configured in ***<font color="#9AAFCB">VRF-2</font>*** and subsequent routing towards the destination.

Ensure that each component is properly configured and verify the traffic flow is going through **Singapore-FW**.

!!! note
    Through this lab, firewall is configured to inspect traffic automatically in **inspect mode**, ***<font color="red"> without requiring any additional configuration</font>***. This inspection ensures that only safe and authorized traffic flows through the network, enhancing security and protecting against potential threats.

## Intended Traffic Flow Diagram

The following diagram illustrates the **<font color="orange">flow of traffic within the network for this scenario</font>**. Traffic is initiated from the **Stockholm-User** and is first redirected to the **Singapore-Firewall** in a **<font color="green">VRF-2</font>** for <font color="orange">**inspection**</font>. After the traffic undergoes inspection, it is then forwarded to the **Sydney-User** in the **Sydney Branch**. 

This scenario demonstrates how traffic is securely routed through the firewall for inspection before reaching its final destination, ensuring that security policies are applied effectively within the SD-WAN fabric.

<figure markdown>
  ![Scenario-2 Traffic Flow](./assets/Scenario-4.gif)
</figure>

## Traffic flow without any policy
In the initial configuration, without applying any traffic policies, the routes learned from the **Sydney-Branch** are distributed equally across both TLOCs, leveraging ECMP (Equal-Cost Multi-Path) for optimal path selection.

```{.ios .no-copy}
Sydney-Branch#show sdwan omp routes 192.168.10.0/24 
Code:
C   -> chosen
I   -> installed
Red -> redistributed
Rej -> rejected
L   -> looped
R   -> resolved
S   -> stale
Ext -> extranet
Inv -> invalid
Stg -> staged
IA  -> On-demand inactive
U   -> TLOC unresolved
BR-R -> Border-Router reoriginated
TGW-R -> Transport-Gateway reoriginated
R-TGW-R -> Reoriginated Transport-Gateway reoriginated

                                                                                                                                                AFFINITY                                 
                                                      PATH                      ATTRIBUTE                                                       GROUP                                    
TENANT    VPN    PREFIX              FROM PEER        ID     LABEL    STATUS    TYPE       TLOC IP          COLOR            ENCAP  PREFERENCE  NUMBER      REGION ID   REGION PATH      
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
0         1      192.168.10.0/24     100.0.0.101      1      1003     C,I,R     installed  10.1.1.1         mpls             ipsec  -           None        None        -                
                                     100.0.0.101      2      1003     C,I,R     installed  10.1.1.1         biz-internet     ipsec  -           None        None        -                

```
To verify this, we initiate a ping from the **Sydney-User** (**<font color="#9AAFCB">IP: 192.168.20.2</font>**) to the **Stockholm-User** (**<font color="#9AAFCB">IP: 192.168.10.2</font>**). A successful ping response confirms that reachability between the two branches is intact.

```{.ios, .no-copy}
Sydney-User:~$ ping 192.168.10.2
PING 192.168.10.2 (192.168.10.2): 56 data bytes
64 bytes from 192.168.10.2: seq=0 ttl=42 time=1.777 ms
64 bytes from 192.168.10.2: seq=1 ttl=42 time=2.463 ms
64 bytes from 192.168.10.2: seq=2 ttl=42 time=2.217 ms
64 bytes from 192.168.10.2: seq=3 ttl=42 time=2.247 ms
^C
--- 192.168.10.2 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 1.777/2.176/2.463 ms
Sydney-User:~$ 
```
Additionally, traffic originating from the **Sydney-Branch** flows directly to the **Stockholm-Branch** via the available TLOCs, ensuring efficient and balanced connectivity in the absence of traffic policies.

```{.ios, .no-copy}
Sydney-User:~$ traceroute  192.168.10.2 -n
traceroute to 192.168.10.2 (192.168.10.2), 30 hops max, 46 byte packets
 1  192.168.20.1  0.416 ms  0.352 ms  0.496 ms
 2  172.16.1.10  0.879 ms  0.635 ms  1.400 ms
 3  192.168.10.2  0.898 ms  1.423 ms  0.852 ms
Sydney-User:~$
```

!!! note
    In the traceroute above, we observe that the traffic is currently routed over the **INET** TLOC. However, it is also possible for the traffic to use the **MPLS** TLOC, as SD-WAN employs ECMP (Equal-Cost Multi-Path) to balance traffic across all available TLOCs.

Following Table exhibit how traffic is flowing from **Sydney-User** to **Stockholm-User**.

| Interface         | IP Address   | Description                                                                                                                          |
|-------------------|--------------|--------------------------------------------------------------------------------------------------------------------------------------|
| GigabitEthernet 3 | 192.168.20.1 | <font color="#9AAFCB"> **Sydney-Branch** WAN-Edge interface in **<font color="orange">VRF 1</font>** connected with **Sydney-User**. |
| GigabitEthernet 1 | 172.16.1.10  | <font color="#9AAFCB"> **Sydney-Branch** WAN-Edge interface **INET TLOC**.</font>                                                    |
| eth0              | 192.168.10.2 | <font color="#9AAFCB"> **Stockholm-User** IP address.</font>                                                                         |

The **Singapore-Branch** WAN-Edge router establishes connectivity with the **Singapore-FW** firewall through its **<font color="#9AAFCB">GigabitEthernet 4</font>** interface, which is part of <font color="orange">**VRF-2**</font>. This interface facilitates the secure and efficient inspection of traffic passing through the firewall. 

The following table provides a detailed overview of the IP addressing configuration assigned to the **Sydney-Branch** WAN-Edge router, ensuring clarity and ease of reference for subsequent tasks in the lab.

| Interface         | IP Address   | Description                                                                            |
|-------------------|--------------|----------------------------------------------------------------------------------------|
| GigabitEthernet 4 | 10.102.102.2 | <font color="#9AAFCB"> **Singapore-FW** GigabitEthernet 4 interface IP address.</font> |


## Configuring Service-Chain in Configuration Group
Next, we will configure a service chain within the service-profile parcel in the configuration group by following the below setps. 

This service chain defines the sequence of services that will be applied to traffic originating from the **Stockholm-Branch** and destined 
for the **Sydney-Branch**. 

By specifying the service chain in the configuration, we instruct the **Stockholm WAN-Edge** on the type of services 
to be applied to the traffic, such as redirection through a **Singapore-FW** firewall in **<font color="bluw">VRF 2</font>**. 

This configuration ensures that the desired service policies are enforced as traffic flows between the branches.

1. From the vManage Landing Page, navigate to the left-hand panel, select Configuration, and click Configuration Groups.
   ![Configuration Group](./assets/S-1-figure-4.png){ .off-glb }
2. Locate and click on the **APAC-Singapore-Branch** Configuration Group as illustrated below.
   ![Locate Configuration Group](./assets/S-4-figure-1.png){ .off-glb }
3. Click the edit ![Edit Icon](./assets/S-1-edit-icon.png){ .off-glb, width=25 } icon for the **APAC-Singapore-Branch - Service Profile** as illustrated below.   
   ![APAC Singapore Configuration Group](./assets/S-4-figure-2.png){ .off-glb } 
4. Click **Add New Feature** and click **Servce Chain Attachment Gateway**.
   ![APAC Singapore Configuration Group](./assets/S-4-figure-3.png){ .off-glb }
5. Click dropdown arrow and select **Add New** in **Service Chain Attachment Gateway** configuration parcel.
   ![APAC Singapore Configuration Group](./assets/S-4-figure-4.png){ .off-glb }
6. Enter the name **Singapore-Firewall-Service-Attachment** and Description **Singapore-Firewall-Service-Attachment** for the service chain attachment gateway.
   ![Service Attachment Gateway Definition](./assets/S-4-figure-5.png){ .off-glb }
7. Now click **<font color="green">Add Service Chain Definition</font>** to add the service chain definition.
   ![Service Attachment Gateway Definition](./assets/S-4-figure-51.png){ .off-glb } 
8. Select name **Singapore-Firewall-SC** and description **Singapore-Firewall-SC**. 
9. Now select **Service Type** <font color="red">**Firewall**</font> by click dropdown and click **<font color="orange">Save</font>**
   ![Service Attachment Gateway Definition](./assets/S-4-figure-14.png){ .off-glb } 
10. Under Basic Information, enter **VPN** <font color="orange">**2**</font>. 
11. Scroll down to **IPv4 Attachment**: <font color="orange">(1 Interface)</font>.
    ![Service Attachment Gateway Definition](./assets/S-1-figure-13.png){ .off-glb } 
12. Enter **Service IPv4 Address <font color="#9AAFCB">10.102.102.2</font>**. This is the IP address of **Singapore Firewall (***<font color="green">Singapore-FW</font>***)**. 
13. Enter SD-WAN Router Interface as **GigabitEthernet4** and click <font color="orange">**Save**</font>.
    ![Service Attachment Gateway Definition](./assets/S-1-figure-14.png){ .off-glb }
    The **GigabitEthernet4** interface on the **Singapore-Branch** WAN-Edge router serves as the connection point for the **Singapore-FW firewall**. 
    This interface facilitates the integration of the firewall into the service chain, allowing traffic to be redirected through the firewall for 
    inspection or policy enforcement as configured. The proper configuration of this interface is crucial for ensuring seamless communication between 
    the WAN-Edge router and the firewall, enabling the desired security and traffic management features within the SD-WAN environment. 
14. Click **Back** at bottom left.
    ![How to go back to Configuration Group](./assets/S-4-figure-7.png){ .off-glb } 
15. As we add the **Service Attachment Gateway Definition**, now configuration group for **Singapore-Sydney-Branch** is now marked as <font color="red">out of sync</font>.
    ![Out of sync](./assets/S-4-figure-6.png){ .off-glb } 
16. Click **APAC-Singapore-Branch** Configuration Group -> Click **<font color="green">Deploy**</font>.
    ![Deoplying Configuration Group with Service Chain Definition](./assets/S-1-figure-16.png){ .off-glb } 
17. In **Deploy Configuration Group** page, select **APAC-Singapore-Branch** by clicking the square Radio Button and Click **Next**.  
    ![Deoplying Configuration Group with Service Chain Definition](./assets/S-4-figure-8.png){ .off-glb } 
18. Click **Import**, and load **<font color="green">APAC-Singapore-Branch.csv</font>** file which loads all the values for the variables.
    ![Attaching CSV file](./assets/S-4-figure-9.png){ .off-glb } 

    !!! info
        All CSV files are located in the **<font color="green">CSV files</font>** folder on the **Desktop** of **_jump-host_**.

19. After uploading the **CSV files**, click **next** then click on **Preview CLI** then select the device from the **left hand** navigation pane to review the configuration changes before deployment. This step ensures that the service-chain gateway definition is correctly included in the configuration. By previewing the CLI, you can verify that all required parameters have been accurately applied and are ready for deployment. This validation step is critical to confirm that the service chain configuration aligns with the intended design and will function as expected once deployed. 
    ![CLI Preview](./assets/S-3-figure-10.png){ .off-glb } 
20. Scroll down the **New Configuration** section to locate the **service-chain number** highlighted in <font color="#9AAFCB">**blue**</font>. <font color="red">Make a note of this number</font>, as it will be required when configuring the data policy in later sections.
    The **service-chain number** is a <font color="red">critical identifier</font> used to link the service chain definition to the appropriate policy, ensuring that traffic is processed through the configured service chain as intended.
    ![CLI Preview of Service Chain Number](./assets/S-4-figure-13.png){ .off-glb } 
21. After finalizing the configuration, click **Cancel** to exit the current screen and then click **Deploy** to initiate the deployment process. Once the deployment is triggered, navigate to the **View Deployment Status** section to monitor the progress. 
    ![CLI Preview of Service Chain Number](./assets/S-2-figure-13.png){ .off-glb } 
22. Wait until the deployment status indicates **<font color="green">Success</font>**, confirming that the configuration has been successfully applied to the relevant devices.
    ![CLI Preview of Service Chain Number](./assets/S-4-figure-11.png){ .off-glb } 
23. To verify the configuration group status, click on the **APAC-Singapore-Branch** configuration group. Ensure that the **Associated row indicates <font color="orange">1</font> device**, confirming that the configuration group is 
    correctly linked to the **Singapore-Branch** WAN-Edge router. Additionally, check that the Provisioning row displays **<font color="orange">0 out of sync</font>** indicating that the configuration has been successfully deployed 
    and is fully synchronized with the device. This step ensures that the configuration group is correctly applied and functioning as intended.
    ![Device is sync.](./assets/S-4-figure-12.png){ .off-glb }

## Verification of Service Chain configuration on Singapore-Branch

In the Cisco SD-WAN architecture, service nodes communicate their available services to the **SD-WAN Controller (vSmart)** using the **Overlay Management Protocol (OMP)** with the service route address family. Each WAN-Edge router is responsible for advertising its service routes to the SD-WAN Controller (vSmart), which then maintains these service routes within its **Routing Information Base (RIB)**. 

**<font color="green">Notably, the SD-WAN Controller (vSmart) controller does not propagate these service routes to other WAN-Edge routers within the SD-WAN fabric</font>**. Instead, the service label, which is advertised by WAN-Edge router in the service route to the SD-WAN Controller (vSmart), plays a crucial role. If traffic destined for a particular vRoute needs to traverse a service, the SD-WAN Controller (vSmart) controller replaces the vRoute’s label with the service label.

```{ .ios, .no-copy, linenums="1", hl_lines="25 26"}
Singapore-Branch#show sdwan omp services 
C   -> chosen
I   -> installed
Red -> redistributed
Rej -> rejected
L   -> looped
R   -> resolved
S   -> stale
Ext -> extranet
Stg -> staged
IA  -> On-demand inactive
Inv -> invalid
BR-R -> Border-Router reoriginated
TGW-R -> Transport-Gateway reoriginated
R-TGW-R -> Reoriginated Transport-Gateway reoriginated

                                                                                 AFFINITY                            
ADDRESS                                                         PATH   REGION    GROUP                               
FAMILY   TENANT    VPN    SERVICE  ORIGINATOR  FROM PEER        ID     ID        NUMBER      LABEL    STATUS    VRF  
---------------------------------------------------------------------------------------------------------------------
ipv4     0         1      VPN      10.0.0.2    0.0.0.0          66     None      None        1008     C,Red,R   1    
                                               0.0.0.0          68     None      None        1008     C,Red,R   1    
         0         2      VPN      10.0.0.2    0.0.0.0          66     None      None        1004     C,Red,R   2    
                                               0.0.0.0          68     None      None        1004     C,Red,R   2    
         0         2      SC6      10.0.0.2    0.0.0.0          66     None      None        1009     C,Red,R   2    
                                               0.0.0.0          68     None      None        1009     C,Red,R   2    
ipv6     0         1      VPN      10.0.0.2    0.0.0.0          66     None      None        1008     C,Red,R   1    
                                               0.0.0.0          68     None      None        1008     C,Red,R   1    
         0         2      VPN      10.0.0.2    0.0.0.0          66     None      None        1004     C,Red,R   2    
                                               0.0.0.0          68     None      None        1004     C,Red,R   2    
```
To verify the service chain configuration on the **Singapore-Branch** WAN-Edge router, access the device CLI and execute the command:

- **show platform software sdwan service-chain database**. 

Review the output to confirm the following details: the **<font color="green">Service Chain ID (e.g., SC6)</font>**, the **<font color="green">VRF (e.g., vrf: 2)</font>**, and the State, which should display **UP** to indicate proper functionality. 

Additionally, verify that the Service is set to **<font color="green">FW (Firewall)</font>**, the TX and RX interface is **GigabitEthernet4**, and the associated IP address is **10.102.102.2**. This verification ensures that the service chain configuration is active and correctly aligned with the intended design.

```{.ios, .no-copy, linenums="1", hl_lines="3 4 5 6 9 17 20" }
Singapore-Branch#show platform software sdwan service-chain database

Service Chain: SC6
   vrf: 2
   label: 1009
   state: up
   description:  Singapore-Firewall-SC

   service: FW
      sequence: 1
      track-enable: true
      state: up
      ha_pair: 1
         type: ipv4
         posture: trusted
         active: [current]
            tx: GigabitEthernet4, 10.102.102.2
                endpoint-tracker: auto
                state: up
            rx: GigabitEthernet4, 10.102.102.2
                endpoint-tracker: auto
                state: up
```

## Routing State Prior to Control Policy Implementation

In an SD-WAN environment, the **SD-WAN controller** maintains routes for each configured **VPN/VRF** and advertises these routes to all 
WAN-Edge routers associated with the **respective VPN/VRF**. This ensures consistent route propagation and connectivity across the network. 
To verify how the **SD-WAN controller** advertises these routes to the WAN-Edge routers, we can use the following command:

- **show omp routes <font color ="orange">vpn 1</font> advertised**
- **show omp routes <font color ="orange">vpn 2</font> advertised**

```{.ios .no-copy linenums="1" hl_lines="1"}
Controller-1# show omp routes vpn 1 advertised
Code:
C   -> chosen
I   -> installed
Red -> redistributed
Rej -> rejected
L   -> looped
R   -> resolved
S   -> stale
Ext -> extranet
Inv -> invalid
Stg -> staged
IA  -> On-demand inactive
U   -> TLOC unresolved

VPN    PREFIX              TO PEER          
--------------------------------------------
1      10.10.10.0/24       10.0.0.1         
                           10.0.0.2         
                           10.1.1.2         
1      10.101.101.0/24     10.0.0.2         
                           10.1.1.1         
                           10.1.1.2         
1      10.102.102.102/32   10.0.0.1         
                           10.1.1.1         
                           10.1.1.2         
1      192.168.10.0/24     10.0.0.1         
                           10.0.0.2         
                           10.1.1.2         
1      192.168.20.0/24     10.0.0.1         
                           10.0.0.2         
                           10.1.1.1         
```
As observed in the output below, the routes from **VRF-1** are <font color="green">**not visible within VRF-2**</font>. 
The displayed prefixes belong exclusively to **VRF-2**, indicating that route-leaking between **VRF-1** and **VRF-2** has not yet been configured or applied. 
This behavior is expected in the **absence of a control policy facilitating inter-VRF route exchange**.

```{.ios .no-copy linenums="1" hl_lines="1"}
Controller-1# show omp routes vpn 2 advertised
Code:
C   -> chosen
I   -> installed
Red -> redistributed
Rej -> rejected
L   -> looped
R   -> resolved
S   -> stale
Ext -> extranet
Inv -> invalid
Stg -> staged
IA  -> On-demand inactive
U   -> TLOC unresolved

VPN    PREFIX              TO PEER          
--------------------------------------------
2      10.20.20.0/24       10.0.0.2         
2      10.102.102.0/24     10.1.1.2         
```
This output highlights the necessity of implementing the **scenario-4** control policy to enable **route-leaking** and achieve full connectivity **between the VRFs**.

Verify the routes in **VRF-1** and **VRF-2** on **Singapore-Branch** WAN-Edge router to confirm routes are not leaked in VRFs.

```{.ios .no-copy linenums="1" hl_lines="1"}
Singapore-Branch#show ip route vrf 1

Routing Table: 1
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, m - OMP
       n - NAT, Ni - NAT inside, No - NAT outside, Nd - NAT DIA
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       H - NHRP, G - NHRP registered, g - NHRP registration summary
       o - ODR, P - periodic downloaded static route, l - LISP
       a - application route
       + - replicated route, % - next hop override, p - overrides from PfR
       & - replicated local route overrides by connected

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
m        10.10.10.0/24 [251/0] via 10.1.1.1, 1d15h, Sdwan-system-intf
m        10.101.101.0/24 [251/0] via 10.0.0.1, 1d17h, Sdwan-system-intf
C        10.102.102.102/32 is directly connected, Loopback1
m     192.168.10.0/24 [251/0] via 10.1.1.1, 1d15h, Sdwan-system-intf
m     192.168.20.0/24 [251/0] via 10.1.1.2, 1d15h, Sdwan-system-intf
```
The following output from the **VRF-2** routing table on **Singapore-Branch** confirms that no routes have been leaked from **VRF-2** into **VRF-1**. 

```{.ios .no-copy linenums="1" hl_lines="1"}
Singapore-Branch#show ip route vrf 2

Routing Table: 2
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, m - OMP
       n - NAT, Ni - NAT inside, No - NAT outside, Nd - NAT DIA
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       H - NHRP, G - NHRP registered, g - NHRP registration summary
       o - ODR, P - periodic downloaded static route, l - LISP
       a - application route
       + - replicated route, % - next hop override, p - overrides from PfR
       & - replicated local route overrides by connected

Gateway of last resort is 0.0.0.0 to network 0.0.0.0

n*Nd  0.0.0.0/0 [6/0], 1d17h, Null0
      10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
m        10.20.20.0/24 [251/0] via 10.1.1.2, 1d15h, Sdwan-system-intf
C        10.102.102.0/24 is directly connected, GigabitEthernet4
L        10.102.102.1/32 is directly connected, GigabitEthernet4
```

## Configuring Centralized Control Policy for Route Leaking

Next, we will configure a centralized **control policy** to ensure that traffic initiated from the **Stockholm-User** destined 
for the **Sydney-User** is first inspected by the **Singapore-FW** before reaching its destination. Since the 
**Singapore-Firewall** (**<font color="green">Singapore--FW</font>**) is positioned within **<font color="orange">VRF-2</font>**, it is necessary to configure a centralized control policy to enable **route-leaking**. This configuration ensures that traffic from the **Stockholm-User** can reach the **Singapore-Firewall** 
and maintain connectivity with the **Sydney-Branch** user in **<font color="orange">VRF-1</font>**. The control policy facilitates seamless 
communication between the two VRFs, ensuring full bidirectional reachability. By implementing this policy, 
we establish a cohesive routing environment that allows resources in **VRF-1** and **VRF-2** to interact efficiently, 
aligning with the network's operational requirements.


!!! info 
    Some **Site-lists**, **VPN-Lists** are **<font color="green">pre-configured</font>**.


1. To begin configuring the centralized data policy, navigate to the left-hand pane in the SD-WAN Manager (vManage) interface. From there, select Configuration, followed by Classic, and then click on Policies. 
   ![Configuring Policies](./assets/S-1-figure-23.png){ .off-glb }
2. Under the Centralized Policy section, click Add Policy to create a new policy. This will initiate the process of defining and implementing the centralized data policy to enforce traffic inspection and routing as per the lab requirements.
   ![Configuring Centralized Policies](./assets/S-1-figure-24.png){ .off-glb }
3. To create the required **Groups of Interest**, start by selecting **Site List** from the left navigation pane within the **Centralized Policy** configuration window. Follow these steps:
   ![Configuring Group of Interests](./assets/S-1-figure-25.png){ .off-glb }
   1. Click Site in left navigation pane and check if following sites are already created. 
      1. Click “**New Site List**” 
         1. Site List Name - **Stockholm-Branch** 
         2. Add Site – <font color="green">10</font>
      2. Click “**New Site List**”
         1. Site List Name - **Sydney-Branch** 
         2. Add Site – <font color="green">20</font>
      3. Click “**New Site List**” 
         1. Site List Name – **Stockholm-Sydney-Singapore** 
         2. Add Site – <font color="green">10,20,102</font> 
![Control Policy Site List](./assets/S-3-figure-14.png){ .off-glb }
   2. Click VPN.  
      1. Click “**New VPN List**” 
         1. VPN List Name – **VPN-1** 
            1. Add VPN – <font color="green">1</font> 
         2. VPN List Name – **VPN-2** 
            1. Add VPN – <font color="green">2</font> 
         3. VPN List Name – **VPN-1-2** 
            1. Add VPN – <font color="green">1,2</font>
![Control Policy VPN List](./assets/S-3-figure-15.png){ .off-glb }
4. Scroll down and click Next.
5. Under Topology, click **<font color="orange">Add Topology</font>** dropdown (for creating route-leaking policy) and select **<font color="green">Custom Control ( Route and TLOC)</font>**.
![Control Policy](./assets/S-3-figure-16.png){ .off-glb }
6. Proceed by entering the required details as outlined below and follow the steps to configure the control policy. 
   * Enter **scenario-4-route-leak** as the policy name. 
   * Description: Provide a brief description, using **scenario-4-route-leak** for clarity and consistency. 
   * Navigate to **<font color="green">Sequence Type</font>**. Under **Add Control Policy**, choose **Route** to define the **route-leaking** configuration.
     These steps ensure that the policy is accurately defined and aligned with the lab's objectives, facilitating effective route-leaking between VPNs as part of the scenario setup.
     ![Control Policy](./assets/S-3-figure-17.png){ .off-glb }
7. Click **<font color="green">Sequence Rule</font>**.
![Control Policy](./assets/S-3-figure-18.png){ .off-glb }
8. In the **Match** section, configure the parameters to define the scope of the **control policy**. Begin by selecting Site and VPN as the matching criteria. Next, specify the following details:
   * **Site List**: Select <font color="orange">**Stockholm-Sydney-Singapore**</font> to include selected branches in the policy.
   * **VPN List**: Choose <font color="orange">**VPN-1-2**</font> to encompass the VPNs involved in the route-leaking configuration.
   This configuration ensures that the policy applies to the specified sites and VPNs, enabling precise control over route-leaking between the designated network segments.
   ![Control Policy](./assets/S-3-figure-19.png){ .off-glb }
9. Next, navigate to the Action section and configure the route-leaking process to enable communication between **VPN-1** and **VPN-2**. 
   Specifically, ensure that routes from **VPN-1** are leaked into **VPN-2** and vice versa. This step is critical to establishing bidirectional
   connectivity, allowing resources in both VPNs to communicate seamlessly. Proper configuration at this stage ensures the integrity of the routing 
   setup and facilitates the intended traffic flow between the VPNs as per the lab topology design.
    1. Click **Action** > **Accept** > **<font color=orange">Export To</font>**. 
    2. Export To: **<font color="orange">VPN-1-2</font>**
![Control Policy](./assets/S-3-figure-20.png){ .off-glb }
10. Click **<font color="orange">Save Match and Actions</font>**.
![Save Control Policy](./assets/S-4-figure-15.png){ .off-glb }
11. Click **Default Action**, click ![pencil](./assets/S-3-figure-pencil.png){ .off-glb width="25"}icon and click **Accept**. Click **<font color="orange">Save Match and Action</font>**. 
![Save Default Action Control Policy](./assets/S-3-figure-22.png){ .off-glb }
12. Click **<font color="green">Save Control Policy</font>** to save the control policy.
13. Now click **Next** and ignore **Configure Traffic Rules** and move to **Apply Policies to Sites and VPNs** section.
![Save Default Action Control Policy](./assets/S-4-figure-16.png){ .off-glb }
14. In order to apply the control policy for route-leaking, select **New Site/WAN Region List** and apply the **<font color="green">scenario-4-route-leak</font>** 
    policy in inbound direction on **Stockholm-Sydney-Singapore** branches. Now after that click **<font color="green">Add and Save Policy</font>**.
    * Enter Policy Name: **scenario-4**.
    * Enter Policy Description: **scenario-4**.
![Applying Control Policy](./assets/S-4-figure-17.png){ .off-glb }
15. Now click ![three dots](./assets/S-1-figure-dots.png){ .off-glb width="25" } and select **Preview** to see the content of the control policy.
![Applying Control Policy](./assets/S-4-figure-18.png){ .off-glb }
16. In order to deploy the policy click **Activate**.
![Activating Control Policy](./assets/S-3-figure-26.png){ .off-glb }
17. Once policy is being pushed successfully, we can have **<font color="green">Success</font>** message that policy is pushed successfully to **SD-WAN controller**.
![Activating Control Policy](./assets/S-3-figure-27.png){ .off-glb }

## Verification of Centralized Control Policy

With the **scenario-4** **<font color="orange">control policy</font>** successfully pushed to facilitate route-leaking between **VPN-1** and **VPN-2**, the next step is to 
verify its implementation. This involves ensuring that the policy has been applied correctly and that the intended routes are being leaked between the two VPNs. 
Additionally, confirm that full reachability is established between **VPN-1** and **VPN-2**, enabling seamless communication as per the lab objectives. 
Verification is a critical step to validate the policy's effectiveness and to ensure that the network behaves as designed.

To ensure that the policy has been applied correctly on the **SD-WAN controller**, the next step involves verifying its implementation through the controller's running configuration. 
By reviewing the running configuration of the control policy, we can confirm that the **scenario-4** policy is correctly defined and operational. This verification process is essential to 
validate the deployment and ensure that the policy is functioning as intended to achieve the desired route-leaking between **VPN-1** and **VPN-2**.

```{ .ios .no-copy linenums="1" hl_lines="1 33"}
Controller-1# show running-config policy
policy
 lists
  vpn-list VPN-1-2
   vpn 1
   vpn 2
  !
  site-list Stockholm-Sydney-Singapore
   site-id 10
   site-id 102
   site-id 20
  !
  prefix-list _AnyIpv4PrefixList
   ip-prefix 0.0.0.0/0 le 32
  !
 !
 control-policy scenario-4-route-leak
  sequence 1
   match route
    prefix-list _AnyIpv4PrefixList
    site-list   Stockholm-Sydney-Singapore
    vpn-list    VPN-1-2
   !
   action accept
    export-to
     vpn-list VPN-1-2
    !
   !
  !
  default-action accept
 !
!
Controller-1# show running-config apply-policy 
apply-policy
 site-list Stockholm-Sydney-Singapore
  control-policy scenario-4-route-leak in
 !
!
```
The output below demonstrates that the **prefix <font color="green">10.102.102.0/24</font>**, which belongs to **VRF-2**, 
has been successfully **leaked into VRF-1**. This confirms that the control policy for route-leaking is now active and 
functioning as intended. The leaked prefix is visible alongside other prefixes **native to VRF-1**, and it is being advertised to the **relevant peers**.

```{.ios .no-copy linenums="1", hl_lines="21 28 29 30 31"}
Controller-1# show omp routes vpn 1 advertised
Code:
C   -> chosen
I   -> installed
Red -> redistributed
Rej -> rejected
L   -> looped
R   -> resolved
S   -> stale
Ext -> extranet
Inv -> invalid
Stg -> staged
IA  -> On-demand inactive
U   -> TLOC unresolved

VPN    PREFIX              TO PEER          
--------------------------------------------
1      10.10.10.0/24       10.0.0.1         
                           10.0.0.2         
                           10.1.1.2         
1      10.20.20.0/24       10.0.0.1         
                           10.0.0.2         
                           10.1.1.1         
                           10.1.1.2         
1      10.101.101.0/24     10.0.0.2         
                           10.1.1.1         
                           10.1.1.2         
1      10.102.102.0/24     10.0.0.1         
                           10.0.0.2         
                           10.1.1.1         
                           10.1.1.2         
1      10.102.102.102/32   10.0.0.1         
                           10.1.1.1         
                           10.1.1.2         
1      192.168.10.0/24     10.0.0.1         
                           10.0.0.2         
                           10.1.1.2         
1      192.168.20.0/24     10.0.0.1         
                           10.0.0.2         
                           10.1.1.1                
```

This validation confirms the effectiveness of the configured control policy in achieving route-leaking, thereby enabling connectivity between the two VRFs as per the lab design.

Prefix **10.102.102.0/24** is now visible in the **VRF-1** routing table on the **Stockholm-Branch**. 

```{.ios .no-copy linenums="1", hl_lines="23 24 25"}
Stockholm-Branch#show ip route vrf 1

Routing Table: 1
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, m - OMP
       n - NAT, Ni - NAT inside, No - NAT outside, Nd - NAT DIA
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       H - NHRP, G - NHRP registered, g - NHRP registration summary
       o - ODR, P - periodic downloaded static route, l - LISP
       a - application route
       + - replicated route, % - next hop override, p - overrides from PfR
       & - replicated local route overrides by connected

Gateway of last resort is 0.0.0.0 to network 0.0.0.0

n*Nd  0.0.0.0/0 [6/0], 1d17h, Null0
      10.0.0.0/8 is variably subnetted, 6 subnets, 2 masks
C        10.10.10.0/24 is directly connected, GigabitEthernet4
L        10.10.10.1/32 is directly connected, GigabitEthernet4
m        10.20.20.0/24 [251/0] via 10.1.1.2, 00:03:45, Sdwan-system-intf
m        10.101.101.0/24 [251/0] via 10.0.0.1, 1d17h, Sdwan-system-intf
m        10.102.102.0/24 [251/0] via 10.0.0.2, 00:03:45, Sdwan-system-intf
m        10.102.102.102/32 [251/0] via 10.0.0.2, 1d17h, Sdwan-system-intf
      192.168.10.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.10.0/24 is directly connected, GigabitEthernet3
L        192.168.10.1/32 is directly connected, GigabitEthernet3
m     192.168.20.0/24 [251/0] via 10.1.1.2, 1d17h, Sdwan-system-intf
```
Let's try to ping from **Stockholm-Branch** WAN-Edge router from **VRF-1** towards **Singapore-FW** ip address **<font color="orange">10.102.102.2</font>**.

```{.ios .no-copy linenums="1", hl_lines="1"}
Stockholm-Branch#ping vrf 1 10.102.102.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.102.102.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/2/3 ms
Stockholm-Branch#
```
Next, verify the routing table on the **Sydney-Branch** WAN-Edge device in **VRF-2** to confirm if the routes from **VRF-1** have been successfully leaked into **VRF-2**. 
This step ensures that the route-leaking configuration is functioning as expected and that traffic originating from **VRF-1** is properly advertised and accessible within **VRF-2**. 
By checking the routing table, we can validate the successful propagation of the routes and confirm that the connectivity between the two VRFs is fully operational.

```{.ios .no-copy linenums="1", hl_lines="1 26 27"}
Sydney-Branch#show ip route vrf 2

Routing Table: 2
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, m - OMP
       n - NAT, Ni - NAT inside, No - NAT outside, Nd - NAT DIA
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       H - NHRP, G - NHRP registered, g - NHRP registration summary
       o - ODR, P - periodic downloaded static route, l - LISP
       a - application route
       + - replicated route, % - next hop override, p - overrides from PfR
       & - replicated local route overrides by connected

Gateway of last resort is 0.0.0.0 to network 0.0.0.0

n*Nd  0.0.0.0/0 [6/0], 1d17h, Null0
      10.0.0.0/8 is variably subnetted, 5 subnets, 2 masks
m        10.10.10.0/24 [251/0] via 10.1.1.1, 00:07:27, Sdwan-system-intf
C        10.20.20.0/24 is directly connected, GigabitEthernet4
L        10.20.20.1/32 is directly connected, GigabitEthernet4
m        10.102.102.0/24 [251/0] via 10.0.0.2, 1d17h, Sdwan-system-intf
m        10.102.102.102/32 [251/0] via 10.0.0.2, 00:07:27, Sdwan-system-intf
m     192.168.10.0/24 [251/0] via 10.1.1.1, 00:07:27, Sdwan-system-intf
m     192.168.20.0/24 [251/0] via 10.1.1.2 (1), 00:07:27, Sdwan-system-intf
```

Verify that we have full reachability from **Stockholm-Branch** towards **<font color="green">Sydney-Branch user</font>** and **<font color="green">Singapore-FW</font>**.

```{.ios .no-copy linenums="1", hl_lines="1 6"}
Stockholm-Branch#ping vrf 1 192.168.20.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.20.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
Stockholm-Branch#ping vrf 1 10.102.102.2 
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.102.102.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/2/4 ms
```
## Configuring Centralized Data Policy for Traffic Steering

Next, we will configure a **centralized data policy** to ensure that traffic initiated from the **Stockholm-User** destined for 
the **Sydney-User** is first inspected by the **<font color="green">Singapore-FW</font>** in **<font color="orange">VRF-2</font>** before reaching its destination. This policy
enforces the required traffic inspection by leveraging the service chain defined earlier. During the configuration, we 
will use the **service-chain number** that was previously configured and noted in **<font color="green">step 20</font>** under "**Configuring Service-Chain in Configuration Group**" section. 
This centralized policy ensures that traffic adheres to the intended security and inspection workflow within the SD-WAN fabric.


1. To begin configuring the centralized data policy, navigate to the left-hand pane in the SD-WAN Manager (vManage) interface. From there, select Configuration, followed by **Classic**, and then click on **Policies**. 
   ![Configuring Policies](./assets/S-1-figure-23.png){ .off-glb }
2. In addition to the previously configured centralized control policy **scenario-4**, where we add **<font color="green">scenario-4-route-leak</font>** for route-leaking, we now introduce a centralized data policy to 
   ensure that traffic is inspected by the **<font color="green">Singapore-FW</font>** in **<font color="green">VRF-2</font>**. This step enhances the traffic management strategy by directing traffic through the firewall 
   for inspection, providing additional security and compliance. To implement this, navigate to the centralized policy section, click on ![dots](./assets/S-1-figure-dots.png){ .off-glb width="25"} next to the <font color="green">**scenario-4**</font> policy, and select **<font color="green">Edit</font>** to add the data policy. 
   This ensures seamless integration of traffic inspection within the existing policy framework.
   ![Configuring Policies](./assets/S-4-figure-25.png){ .off-glb }
3. In order to add data policy, click **Traffic Rules > Traffic Data**.
   ![Configuring Policies](./assets/S-3-figure-29.png){ .off-glb }
4. Now click **Add Policy** and then click **Create New**.
   ![Configuring Policies](./assets/S-3-figure-30.png){ .off-glb }
5. Follow the below steps to start configuring data policy.
   * Enter Name – **scenario-4-data-policy**
   * Description - **scenario-4-data-policy**
6. Now click ![pencil](./assets/S-3-figure-pencil.png){ .off-glb width="25"} icon in “**<font color="green">Default Action</font>**”
   * Under “**<font color="orange">Actions</font>**” Select **<font color="green">Accept</font>**. 
   * Click “**Save and Match**”.
     ![Configuring Data Policies](./assets/S-3-figure-31.png){ .off-glb }
7. Click **Sequence Type**.
   ![Configuring Data Policies](./assets/S-3-figure-32.png){ .off-glb }
8. From “**Add Data Policy**” pop-up, Select **<font color="green">Custom</font>**.
   ![Configuring Data Policies](./assets/S-3-figure-33.png){ .off-glb }
9. Click “**Sequence Rule**”.
   ![Configuring Data Policies](./assets/S-3-figure-34.png){ .off-glb }
10. **Match** > **Scroll** right to select and click **Source Data Prefix**.
    ![Configuring Data Policies](./assets/S-3-figure-35.png){ .off-glb }
11. Under “**Match Conditions**”. 
    * Click in box with “**Source Data Prefix List**” and **select** > **<font color="green">Stockholm-Branch-User</font>**.
13. Under “**Match Conditions**”. 
    * Scroll down and click in box with “**Destination Data Prefix List**” and **select** > **<font color="green">Sydney-Branch-User</font>**.
    ![Configuring Data Policies](./assets/S-4-figure-23.png){ .off-glb }
14. Scroll up and select “**<font color="green">Actions</font>**”.
    * Click **Accept** Radio button. 
    ![Configuring Data Policies](./assets/S-3-figure-37.png){ .off-glb }
    * Scroll to the right to select “**<font color="green">Service Chain</font>**”.
    ![Configuring Data Policies](./assets/S-4-figure-24.png){ .off-glb }
15. Click **<font color="orange">Service Chain Type</font>** and scroll the options down a bit and select “**Service Chain Type**” – for example **<font color="green">SC6</font>**. 
    - Under VPN, specify VPN **<font color="green">2</font>**.
    - Select **<font color="orange">Remote</font>**.
    - Under **TLOC section** enter the following information:
      - **IP** – **<font color="green">10.0.0.2</font>**.
      - **Color** – **<font color="green">biz-internet</font>**.
      - **Encapsulation** - **<font color="green">IPSEC</font>** 
  - Uncheck **<font color="green">Restrict</font>**. 
  ![Configuring Data Policies](./assets/S-4-figure-19.png){ .off-glb }

    !!! info
        When **restrict** is configured in the set service-chain action, packets are dropped if a service chain goes down or if the **TLOCs** that are specified in a policy are **NOT** available. The restrict behavior is suitable for security services such as a <font color="green">firewall</font>.

    !!! warning
        Use the <font color="red">**Service Chain Type**</font> from **point 20** of "**Configuring Service-Chain in Configuration Group**".

16. Click “**Save Match and Actions**”. 
    ![Configuring Data Policies](./assets/S-4-figure-20.png){ .off-glb }
17. Once data policy is saved, we can click **Policy Application > Traffic Data**, we select **<font color="green">Traffic Data</font>** to apply the data policy **scenario-4-data-policy**.
    ![Configuring Data Policies](./assets/S-4-figure-26.png){ .off-glb }
18. Now Click **New Site/WAN Region List and VPN List**
    * Keep **<font color="orange">From Service</font>** radio button checked.
    * Keep **<font color="orange">Site List</font>** radio button checked.
    * Select Site List by clicking in the box – **<font color="green">Stockholm-Branch</font>**
    * Click outside the selection box to expose **Select VPN List**.
    * Select VPN List by clicking in the box – **<font color="green">VPN-1</font>**. 
    ![Configuring Data Policies](./assets/S-4-figure-21.png){ .off-glb }
19. Click **<font color="orange">Add and Save Policy Changes</font>** at the bottom.
21. Click **Activate** on **Activate Policy** pop-up.
![Configuring Data Policies](./assets/S-3-figure-44.png){ .off-glb }
22. Once policy is being pushed successfully, we can have **Push vSmart Policy** **Validation success** and **Message** “**<font color="green">Done – Push vSmart Policy**</font>”. 
    ![Configuring Data Policies](./assets/S-3-figure-45.png){ .off-glb }

## Verification

After the centralized data policy has been successfully deployed, the next step is to confirm that the policy has been 
propagated by the SD-WAN controller (vSmart) to the WAN-Edges. In this case, we need to ensure that the **Stockholm-Branch** WAN-Edge 
has received the policy via OMP and is correctly steering traffic through the **Singapore-FW** in **<font color="orange">VRF-2</font>** as intended.

To verify this, we can utilize the following show command on the **Stockholm-Branch** WAN-Edge. This will help confirm whether the 
centralized data policy has been effectively pushed from the SD-WAN controller (vSmart) to the **Stockholm-Branch** router through OMP.

```{ .ios .no-copy linenums="1", hl_lines="1" }
Stockholm-Branch#show sdwan policy from-vsmart 
from-vsmart data-policy _VPN-1_scenario-4-data-policy
 direction from-service
 vpn-list VPN-1
  sequence 1
   match
    source-data-prefix-list      Stockholm-Branch-User
    destination-data-prefix-list Sydney-Branch-User
   action accept
    set
     vpn-label 8389617
     service-chain SC6
     service-chain vpn 2
     service-chain fall-back
     service-chain tloc 10.0.0.2
     service-chain tloc color biz-internet
     service-chain tloc encap ipsec
  default-action accept
from-vsmart lists vpn-list VPN-1
 vpn 1
from-vsmart lists data-prefix-list Stockholm-Branch-User
 ip-prefix 192.168.10.0/24
from-vsmart lists data-prefix-list Sydney-Branch-User
 ip-prefix 192.168.20.0/24
```
!!! info
    In our centralized policy configuration, we have implemented both a control policy and a data policy. Within the Cisco SD-WAN policy framework, 
    it is important to note that centralized control policies are processed directly on the **SD-WAN controller** and are not pushed to the WAN-Edge devices. 
    Conversely, **centralized data policies** are distributed to the WAN-Edge devices for local enforcement. As a result, when inspecting the configuration 
    on the **Stockholm-Branch** WAN-Edge, only the data policy will be visible. This distinction ensures that control decisions are managed centrally while data traffic 
    is handled locally at the edge for optimized performance and enforcement.


!!! warning
    It could be possible that Service-chain number might be different.

To verify that the centralized data policy is functioning as intended, navigate back to the **Sydney-User** in the **Sydney-Branch** site. 

- Perform a traceroute to the **Sydney-User** located in the **Sydney-Branch** site using the **traceroute** command: 
    - _traceroute 192.168.20.2 -n_
- Observe the traceroute output to confirm that traffic is hitting the **Singapore firewall (Singapore-FW)** at IP address **<font color="#9AAFCB">10.102.102.2</font>**, which is in **<font color="green">VRF-2</font>**.

```{.ios .no-copy linenums="1", hl_lines="3"}
Stockholm-User:~$ traceroute 192.168.20.2 -n
traceroute to 192.168.20.2 (192.168.20.2), 30 hops max, 46 byte packets
 1  192.168.10.1  1.004 ms  0.638 ms  0.556 ms
 2  172.16.1.102  1.222 ms  0.960 ms  1.178 ms
 3  10.102.102.2  2.540 ms  1.872 ms  2.170 ms
 4  172.16.1.102  2.236 ms  3.146 ms  2.701 ms
 5  172.16.2.20  3.736 ms  3.367 ms  3.102 ms
 6  192.168.20.2  3.104 ms  3.389 ms  3.177 ms
Stockholm-User:~$ 
```
- Next, verify on the **Singapore-FW** itself to ensure that the traffic is being **inspected** before continuing its journey toward the **Sydney-User**. 
- This step confirms that the traffic is correctly following the service chain configuration as defined in the centralized policy **<font color="green">even though users are in different VRF</font>**.

```{.ios .no-copy title="Singapore Firewall traffic inspection"}
Singapore-FW# show conn all
12 in use, 13 most used

UDP inside  192.168.10.2:34198 inside  192.168.20.2:33441, idle 0:00:01, bytes 0, flags - 
UDP inside  192.168.10.2:34198 inside  192.168.20.2:33445, idle 0:00:01, bytes 18, flags - 
UDP inside  192.168.10.2:34198 inside  192.168.20.2:33447, idle 0:00:01, bytes 18, flags - 
UDP inside  192.168.10.2:34198 inside  192.168.20.2:33449, idle 0:00:01, bytes 18, flags - 
UDP inside  192.168.10.2:34198 inside  192.168.20.2:33442, idle 0:00:01, bytes 0, flags - 
UDP inside  192.168.10.2:34198 inside  192.168.20.2:33446, idle 0:00:01, bytes 18, flags - 
UDP inside  192.168.10.2:34198 inside  192.168.20.2:33444, idle 0:00:01, bytes 18, flags - 
UDP inside  192.168.10.2:34198 inside  192.168.20.2:33452, idle 0:00:01, bytes 18, flags - 
UDP inside  192.168.10.2:34198 inside  192.168.20.2:33451, idle 0:00:01, bytes 18, flags - 
UDP inside  192.168.10.2:34198 inside  192.168.20.2:33448, idle 0:00:01, bytes 18, flags - 
UDP inside  192.168.10.2:34198 inside  192.168.20.2:33443, idle 0:00:01, bytes 0, flags - 
UDP inside  192.168.10.2:34198 inside  192.168.20.2:33450, idle 0:00:01, bytes 18, flags - 
Singapore-FW# 
```
We can use the following show commands to see packets are matching and service chaining is working on **Singapore** WAN-Edge router. 

```{.ios .no-copy linenums="1", hl_lines="1 3 4 5 6 9"}
Singapore-Branch#show platform software sdwan service-chain stats detail 

Service Chain: SC6
   vrf: 2
   label: 1009
   state: up
   description:  Singapore-Firewall-SC

   service: FW
      tx: 12 rx: 9
      ha_pair 1: ipv4
         active
            tx: 12 rx: 9 
            tx tracker: sent: 4 dropped: 0 rtt: 1
            rx tracker: sent: 0 dropped: 0 rtt: 0
         backup
            tx: 0 rx: 0 
            tx tracker: sent: 0 dropped: 0 rtt: 0
            rx tracker: sent: 0 dropped: 0 rtt: 0
```

```{.ios .no-copy linenums="1", hl_lines="1 2 7 8"}
Stockholm-Branch#show platform hardware qfp active feature sdwan datapath service-chain stats      
Service-Chain ID: 6
  Global stats: 12
  Global stats v6: 0
  Per Service stats 
    Service: Firewall
      Tx pkt: 12
      Rx pkt: 9
      Tx pkt v6: 0
      Rx pkt v6: 0 
```
## Conclusion
In conclusion, the configuration group and centralized policy implemented in this lab successfully achieved the intended traffic 
flow and inspection requirements. Traffic originating from the **Stockholm-User** at **Stockholm-Branch (site-10)** and destined for the **Sydney-User** at **Sydney-Branch (site-20)** 
was effectively routed through the **Singapore (Singapore-FW)** located at **Singapore-Branch** in a **separate VRF (VRF-2)**. The centralized control policy **enabled route-leaking between VRF-1 and VRF-2**, allowing the **Stockholm-Branch** WAN-Edge to establish connectivity with the firewall. 
The centralized data policy ensured that all traffic was directed through the **Singapore-firewall** for inspection before proceeding to its destination. This demonstrates the effective use of service chaining and centralized policy mechanisms in Cisco SD-WAN to implement advanced traffic steering and security measures.

!!! info
    Before proceeding to the **next lab**, it is essential to **<font color="red">deactivate</font>** the centralized policy configured in the current exercise. **Deactivating** the policy 
    ensures that no unintended traffic steering or service chaining configurations remain active, which could interfere with subsequent lab tasks. Once the centralized policy is successfully deactivated 
    and confirmed, delete the service chain parcel **Singapore-Branch-Service-Attachment** in **Singapore-Branch** WAN-Edge service profile **APAC-Singapore-Branch-Service-VPN**. Now we can confidently move forward to 
    the next lab. This step is critical to maintain a clean and controlled environment for the upcoming configurations and scenarios.
    From the left hand navigation pane select **<font color="red">Configuration -> Policies -> {{Respected-Scenario-Centralized-Policy}} -> ... -> Deactivate</font>**.