# Routing Traffic Through Firewall in Singapore Branch Across VRFs for Site-to-Site Communication

## Introduction

In this lab exercise, you will analyze the traffic flow between a user in the **<font color="Green">APAC-Sydney-Branch (site-20)</font>** and a user in the **<font color="Green">EMEA-Stockholm-Branch (site-10)</font>**, with the traffic routed through a **firewall** **<font color="Green">(Sydney-Firewall)</font>** hosted at the 
**APAC-Sydney (site-20)** in a different **<font color="#9AAFCB">VRF-2</font>**. The **Sydney-Firewall**, accessible via **Sydney-Branch** WAN-Edge router in **<font color="#9AAFCB">VRF-2</font>**, plays a critical role in inspecting and securing the traffic as it traverses the network. This exercise will focus on understanding the configuration and verification of service chaining, 
centralized policies, and the interactions between network elements to ensure the intended traffic flow through the designated security device.

Here is a breakdown of the key components involved in the network path:

- **Source:** The traffic originates from a **Stockholm-User** in the **EMEA-Stockholm-Branch (site-10)**.
- **Destination:** The intended recipient is a **Sydney-User** in the A**PAC-Sydney-Branch (site-20)**.
- **Firewall:** All traffic passes through a **firewall (Singapore-FW)**, which is hosted locally at the **APAC-Singapore-Branch (site-20)** in **<font color="green">VRF-2</font>**.
- **WAN Edge Router:** The **Singapore-Branch** WAN-Edge router, facilitates the traffic's reachability to the firewall configured in ***<font color="#9AAFCB">VRF-2</font>*** and subsequent routing towards the destination.

Ensure that each component is properly configured and verify the traffic flow is going through **Singapore-FW**.

!!! note
    Through this lab, firewall is configured to inspect traffic automatically in **inspected mode**, ***<font color="red"> without requiring any additional configuration</font>***. This inspection ensures that only safe and authorized traffic flows through the network, enhancing security and protecting against potential threats.

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
for the **Sydney-Branch**. By specifying the service chain in the configuration, we instruct the **Stockholm WAN-Edge** on the type of services 
to be applied to the traffic, such as redirection through a **Singapore-FW** firewall in **<font color="bluw">VRF 2</font>**. 
This configuration ensures that the desired service policies are enforced as traffic flows between the branches.

1. From the vManage Landing Page, navigate to the left-hand panel, select Configuration, and click Configuration Groups.
   ![Configuration Group](./assets/S-1-figure-4.png){ .off-glb }
2. Locate and click on the **APAC-Singapore-Branch** Configuration Group as illustrated below.
   ![Locate Configuration Group](./assets/S-4-figure-1.png){ .off-glb }
3. Click the edit ![Edit Icon](./assets/S-1-edit-icon.png){ .off-glb, width=25 } icon for the **APAC-Singapore-Branch - Service Profile** as illustrated below.   
   ![APAC Singapore Configuration Group](./assets/S-4-figure-2.png){ .off-glb }
4. Click ![pencil](./assets/S-1-figure-pencil.png){ .off-glb } icon of **APAC-Singapore-Branch-Service Profile** and click **Add New Feature** and click **Servce Chain Attachment Gateway**.
   ![APAC Singapore Configuration Group](./assets/S-4-figure-3.png){ .off-glb }
5. Click dropdown arrow and select **Add New** in **Service Chain Attachment Gateway** configuration parcel.
   ![APAC Singapore Configuration Group](./assets/S-4-figure-4.png){ .off-glb }
6. Enter the name **Singapore-Firewall-Service-Attachment** and Description **Singapore-Firewall-Service-Attachment** for the service chain attachment gateway.
   ![Service Attachment Gateway Definition](./assets/S-4-figure-5.png){ .off-glb }
7. Now click **<font color="green">Add Service Chain</font>** to add the service chain definition.
   ![Service Attachment Gateway Definition](./assets/S-4-figure-4.5.png){ .off-glb }
7. Select name **Singapore-Firewall-SC-Def** and description **Singapore-Firewall-SC-Def**.
8. Now select **Service Type** <font color="red">**Firewall**</font> by click dropdown and click **<font color="orange">Save</font>**
   ![Service Attachment Gateway Definition](./assets/S-3-figure-6.png){ .off-glb } 
9. Under Basic Information, enter **VPN** <font color="orange">**2**</font>. 
10. Scroll down to **IPv4 Attachment**: <font color="orange">(1 Interface)</font>.
    ![Service Attachment Gateway Definition](./assets/S-1-figure-13.png){ .off-glb } 
11. Enter **Service IPv4 Address <font color="#9AAFCB">10.102.102.2</font>**. This is the IP address of **Singapore Firewall (***<font color="green">Singapore-FW</font>***)**. 
12. Enter SD-WAN Router Interface as **GigabitEthernet4** and click <font color="orange">**Save**</font>.
    ![Service Attachment Gateway Definition](./assets/S-1-figure-14.png){ .off-glb }
    The **GigabitEthernet4** interface on the **Singapore-Branch** WAN-Edge router serves as the connection point for the **Singapore-FW firewall**. 
    This interface facilitates the integration of the firewall into the service chain, allowing traffic to be redirected through the firewall for 
    inspection or policy enforcement as configured. The proper configuration of this interface is crucial for ensuring seamless communication between 
    the WAN-Edge router and the firewall, enabling the desired security and traffic management features within the SD-WAN environment. 
13. Click **Back** at bottom left.
    ![How to go back to Configuration Group](./assets/S-3-figure-7.png){ .off-glb } 
14. As we add the **Service Attachment Gateway Definition**, now configuration group for **Singapore-Sydney-Branch** is now marked as <font color="red">out of sync</font>.
    ![Out of sync](./assets/S-4-figure-6.png){ .off-glb } 
