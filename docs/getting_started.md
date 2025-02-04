# Getting Started
Connecting to the lab network requires VPN access to the lab environment. After successfully establishing VPN connection, you access with RDP the jump-host workstation with preconfigured session manager.

> :memo: **How to RDP**
>
> - RDP to the Management Workstation.
> - Open the Remote Desktop Connection application
> ![SD-WAN Topology](./assets/RDP.png)

## Understanding the Lab Topology
This lab features a topology comprising four WAN-Edge routers strategically deployed across two regions: **EMEA** and **APAC**. In the **EMEA region**, the routers are located in **London** and **Stockholm**, while in the **APAC region**, they are deployed in **Singapore** and **Sydney**. Each WAN-Edge router is integrated with a <font color="red">**firewall**</font>, which is essential for redirecting traffic for inspection across various use cases. The deployment leverages two transport types ***Internet*** and ***MPLS*** to provide reliable connectivity between the WAN-Edge routers across regions. This setup creates a robust environment for simulating real-world traffic routing and inspection scenarios.

## Device Connectivity and Interfaces

In the lab topology, each WAN-Edge router is configured with dual transport connectivity: the **GigabitEthernet 1** interface is connected to the **Internet transport**, while the **GigabitEthernet 2** interface is connected to the **MPLS transport**. For simplification, the deployment includes a single SD-WAN validator (vBond) and a single SD-WAN controller (vSmart), each assigned the following **system ip** mentioned in below table.

| Device               | System IP   |
|----------------------|-------------|
| Validator  (vBond)   | 100.0.0.201 |
| Controller (vSmart)  | 100.0.0.101 |
| Manager    (vManage) | 100.0.0.1   |

Additionally, the **GigabitEthernet 4** interface on each region WAN-Edge router is connected to a <font color="red">**firewall**</font> within the service VPN. <font color="orange">***Branch users***</font> are also part of the service VPN and are connected to the WAN-Edge routers through the **GigabitEthernet 3** interface, as shown in the topology diagram. This setup enables seamless traffic flow and inspection while ensuring a realistic representation of enterprise network environments.

Each WAN-Edge router is uniquely identified by a **system ip** address, as detailed in the table below. These system IPs are critical for establishing secure communication and management within the SD-WAN fabric, enabling streamlined routing, control, and policy enforcement across the network. The table provides a clear mapping of each WAN-Edge router to its respective system IP address for reference throughout the lab.

| Device             | System IP |
|:-------------------|-----------|
| London    WAN-Edge | 10.0.0.1  |
| Singapore WAN-Edge | 10.0.0.2  |
| Stockholm WAN-Edge | 10.1.1.1  |           
| Sydney    WAN-Edge | 10.1.1.2  |


## Topology diagram

<figure markdown>
  ![SD-WAN Topology](./assets/sdwan-topology.png)
</figure>

## Ensuring Proper Configuration Using Configuration Groups

In this lab, the WAN-Edge routers will be provisioned using **configuration groups (CG)**, while the **SD-WAN Controller (vSmart)** is managed through **template**. This approach ensures that the **SD-WAN controller (vSmart)** remains fully integrated with the SD-WAN Manager (vManage), enabling the seamless creation and deployment of various policies later in the lab. 

The table below lists the configuration group names assigned to each WAN-Edge router. It is essential to verify that each WAN-Edge router is correctly attached to its designated configuration group to ensure proper functionality.

| **Device**       | **Configuration Group Name** | **System IP** |
|------------------|------------------------------|---------------|
| London-Branch    | EMEA-London-Branch           | 10.0.0.1      |
| Stockholm-Branch | EMEA-Stockholm-Branch        | 10.1.1.1      |
| Singapore-Branch | APAC-Singapore-Branch        | 10.0.0.2      |
| Sydney-Branch    | APAC-Sydney-Branch           | 10.1.1.2      |


To confirm the configuration, you can use the <font color="orange">**show sdwan system status</font>** command on each WAN-Edge router. This command provides details on the configuration group attached to the device, allowing you to validate the setup.

!!! note
    The show outputs provided below are from the **London-Branch**; however, it is important to perform this verification on each WAN-Edge router in the topology.


``` {.ios, .no-copy, title="Configuration Template Verification", linenums="1", hl_lines="38 40"}
London-Branch#show sdwan system status
Viptela (tm) vEdge Operating System Software
Copyright (c) 2013-2024 by Viptela, Inc.
Controller Compatibility: 20.15
Version: 17.15.01a.0.193
Build: Not applicable

System logging to host  is disabled
System logging to disk is enabled

System state:            GREEN. All daemons up
System FIPS state:       Disabled

Last reboot:             reload. 
CPU-reported reboot:     Initiated by other
Boot loader version:     Not applicable
System uptime:           9 days 19 hrs 42 min 56 sec
Current time:            Thu Dec 19 11:15:28 UTC 2024

Hypervisor Type:         KVM
Cloud Hosted Instance:   false

Load average:            1 minute: 1.69, 5 minutes: 1.65, 15 minutes: 1.53
Processes:               358 total
CPU allocation:          4 total,   1 control,   3 data
CPU states:              20.06% user,   8.30% system,   71.61% idle
Memory usage:            4985676K total,    3488952K used,   1496724K free
                         25700K buffers,  1645468K cache

Disk usage:              Filesystem      Size   Used  Avail   Use %  Mounted on
                         /dev/disk/by-label/fs-bootflash       4933M  1312M  3350M   28%   /bootflash
                                387M  165M  217M   43   /bootflash/.sdwaninstaller

Personality:             vEdge
Model name:              C8000V
Device role:             cEdge-SDWAN
Services:                None
vManaged:                true
Commit pending:          false
Configuration template:  EMEA-London-Branch
Chassis serial number:   SSI130300YK
```
If any WAN-Edge router is **not managed by the SD-WAN Manager (vManage)** or does not have a configuration group attached, you can log in to SD-WAN Manager (vManage) and confirms if WAN-Edge router is associated with configuration group or not.

!!! info
    **SD-WAN Manager(vManage)** is configured with the username and password combination (**ltrent2008**/**ltrent2008**).

You can follow the path once you logged into SD-WAN Manager by clicking **<font color="green">Configuration -> Configuration Groups</font>**.

<figure markdown>
  ![cg](./assets/Configuration-Group.png){ width="300", align=left, .off-glb }
</figure>

Verify that corresponding configuration group have a device associated with it or not like below.

<figure markdown>
  ![Configuration Group Association](./assets/CG-Association.png){ align=left, .off-glb }
</figure>

If the WAN-Edge router is not associated with the configuration group, then we can click **Add**.

<figure markdown>
  ![Configuration Group Add Device](./assets/cg-add.png){ width="300", align=left, .off-glb }
</figure>

One we see list of WAN-Edge devices, we need to make sure we select the right device as per the configuration group name.
As in our exhibit below, we selected **EMEA-Stockholm-Branch**

<figure markdown>
  ![Configuration Device Selection](./assets/cg-associate-selection.png){ align=left, .off-glb }
</figure>

To populate the values for the variables used in the configuration group, you can utilize the **import** feature in the SD-WAN Manager (vManage). 
Navigate to the appropriate section for the configuration group, click ***import***, and locate the respective CSV file for each device. 
These files are conveniently stored on the desktop of the workstation. Importing the CSV file will automatically fill in the required variable values, ensuring the configuration group is deployed successfully and accurately to the intended devices.

<figure markdown>
  ![Attaching CSV File to Configuration Group](./assets/Attaching-CSV-To-CG.png){ align=left, .off-glb }
</figure>

After successfully importing the CSV file, it is important to verify the configuration before deployment. You can do this by clicking **Preview CLI** in the SD-WAN Manager (vManage). This feature allows you to review the CLI commands generated based on the imported variables, providing a clear view of the changes that will be applied to the device. 
Reviewing the CLI output ensures accuracy and helps identify any potential errors before proceeding. Once satisfied with the configuration, you can confidently click **Deploy** to apply the settings to the device.

<figure markdown>
  ![Preview CLI](./assets/Preview-CLI.png){ align=left, .off-glb }
</figure>

After the configuration group is successfully deployed, the SD-WAN Manager (vManage) will display a **Success** notification. This confirmation indicates that the configuration has been applied without errors and is now active on the device. The notification serves as a validation step, ensuring that the deployment process was completed successfully and that the device is fully integrated with the intended settings within the SD-WAN fabric.

<figure markdown>
  ![Configuration Group Deployment Succesfully](./assets/CG-Success.png){ align=left, .off-glb }
</figure>

## Lab Connectivity

To verify the SD-WAN fabric's control connections, we can utilize following show commands to confirm that each WAN-Edge router has established a control connection with the SD-WAN controllers, including the **SD-WAN Manager (vManage, System IP: 100.0.0.1)** and the **SD-WAN Controller (vSmart, System IP: 100.0.0.101)**. Additionally, it is crucial to validate that OMP (Overlay Management Protocol) peering is established with the SD-WAN Controller, as this is essential for route exchange and policy enforcement.

- show sdwan control local-properties
- show sdwan control connections
- show sdwan omp peers
- show sdwan bfd sessions table
- show sdwan tunnel statistics table 

!!! info
    Each WAN-Edge router in the topology is configured with the default username and password combination (**admin**/**admin**).


Initially, the WAN-Edge routers are configured for full mesh connectivity. Due to the use of the **restrict** option under the TLOC configuration, IPSec tunnels and BFD sessions are established only among TLOCs with matching colors. As a result, each WAN-Edge router will establish a total of **6 IPSec tunnels and 6 BFD sessions**, ensuring robust and efficient connectivity within the SD-WAN fabric.


## Verifying Lab Connectivity

The first step is to verify that the certificates are correctly installed on each WAN-Edge router, as they are essential for establishing secure communication within the SD-WAN fabric. 

!!! note
    The show outputs provided below are from the **Stockholm Branch**; however, it is important to perform this verification on each WAN-Edge router in the topology.

``` { .ios, .no-copy }
Stockholm-Branch#show sdwan control local-properties 
personality                       vedge
sp-organization-name              cml-sdwan-lab-tool
organization-name                 cml-sdwan-lab-tool
root-ca-chain-status              Installed
root-ca-crl-status                Not-Installed

certificate-status                Installed
certificate-validity              Valid
certificate-not-valid-before      Nov 26 14:23:05 2024 GMT
certificate-not-valid-after       Nov 24 14:23:05 2034 GMT

enterprise-cert-status            Not Applicable
enterprise-cert-validity          Not Applicable
enterprise-cert-not-valid-before  Not Applicable
enterprise-cert-not-valid-after   Not Applicable

dns-name                          validator.sdwan.local
site-id                           10
domain-id                         1
protocol                          dtls
tls-port                          0
system-ip                         10.1.1.1
chassis-num/unique-id             C8K-3D1A8960-6E76-532C-DA93-50626FC5797E
serial-num                        42F4AE15
subject-serial-num                N/A
enterprise-serial-num             No certificate installed
token                             Invalid
keygen-interval                   1:00:00:00
retry-interval                    0:00:00:17
no-activity-exp-interval          0:00:00:20
dns-cache-ttl                     0:00:00:00
port-hopped                       TRUE
time-since-last-port-hop          13:12:53:22
embargo-check                     success
device-role                       edge-router
region-id-set                     N/A
mrf-migration-mode                disabled
mrf-management-region             no
number-vbond-peers                1

INDEX   IP                                      PORT
----------------------------------------------------
0       172.16.0.201                            12346  

number-active-wan-interfaces      2

          
 NAT TYPE: E -- indicates End-point independent mapping
           A -- indicates Address-port dependent mapping
           N -- indicates Not learned
           Note: Requires minimum two vbonds to learn the NAT type

                         PUBLIC          PUBLIC PRIVATE         PRIVATE                                 PRIVATE                        WAN   MAX   RESTRICT/           LAST         SPI TIME    NAT  VM          BIND
INTERFACE                IPv4            PORT   IPv4            IPv6                                    PORT    VS/VM COLOR            STATE CNTRL CONTROL/     LR/LB  CONNECTION   REMAINING   TYPE CON REG     INTERFACE
                                                                                                                                                   STUN                                              PRF IDs
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
GigabitEthernet1              172.16.1.10     12386  172.16.1.10     ::                                      12386    1/1  biz-internet     up     2     yes/yes/no   No/No  0:00:00:00   0:11:17:15  N    5  Default N/A                           
GigabitEthernet2              172.16.2.10     12386  172.16.2.10     ::                                      12386    1/0  mpls             up     2     yes/yes/no   No/No  0:00:00:00   0:11:17:15  N    5  Default N/A                           
```

Additionally, ensure that all interfaces are operational and in an "up" state. This validation is critical to confirm that the devices are properly configured and ready for subsequent tasks in the lab.

```{ .ios, .no-copy}
Stockholm-Branch#show ip interface brief 
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       172.16.1.10     YES other  up                    up      
GigabitEthernet2       172.16.2.10     YES other  up                    up      
GigabitEthernet3       192.168.10.1    YES other  up                    up      
GigabitEthernet4       10.10.10.1      YES other  up                    up      
GigabitEthernet5       unassigned      YES unset  up                    up      
GigabitEthernet6       unassigned      YES unset  up                    up      
GigabitEthernet7       unassigned      YES unset  up                    up      
GigabitEthernet8       unassigned      YES unset  up                    up      
Sdwan-system-intf      10.1.1.1        YES unset  up                    up      
vmanage_system         unassigned      YES unset  up                    up      
Loopback65528          192.168.1.1     YES other  up                    up      
Loopback65529          11.1.1.1        YES other  up                    up      
NVI0                   unassigned      YES unset  up                    up      
Tunnel1                172.16.1.10     YES TFTP   up                    up      
Tunnel2                172.16.2.10     YES TFTP   up                    up      
Stockholm-Branch#
```

Next, verify that the control connections are successfully established with the SD-WAN Manager (vManage) and the SD-WAN Controller (vSmart) on each WAN-Edge device. This step ensures that the WAN-Edge routers are fully integrated into the SD-WAN fabric. 

```{ .ios, .no-copy}
Stockholm-Branch#show sdwan control connections
                                                                                          PEER                                          PEER                                          CONTROLLER
PEER    PEER    PEER            SITE       DOMAIN PEER                                    PRIV  PEER                                    PUB                                           GROUP
TYPE    PROT    SYSTEM IP       ID         ID     PRIVATE IP                              PORT  PUBLIC IP                               PORT  ORGANIZATION            LOCAL COLOR     PROXY STATE UPTIME      ID
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
vsmart  dtls    100.0.0.101     100        1      172.16.0.101                            12346 172.16.0.101                            12346 cml-sdwan-lab-tool      biz-internet    No    up     0:00:45:19 0           
vsmart  dtls    100.0.0.101     100        1      172.16.0.101                            12346 172.16.0.101                            12346 cml-sdwan-lab-tool      mpls            No    up     0:00:45:19 0           
vmanage dtls    100.0.0.1       100        0      172.16.0.1                              12746 172.16.0.1                              12746 cml-sdwan-lab-tool      biz-internet    No    up     0:00:45:19 0           
```

Additionally, confirm that OMP (Overlay Management Protocol) peering is active on each WAN-Edge router, as this is critical for route exchange and the implementation of SD-WAN policies.

```{ .ios, .no-copy}
Stockholm-Branch#show sdwan omp peers 
R -> routes received
I -> routes installed
S -> routes sent

TENANT                             DOMAIN    OVERLAY   SITE      REGION                                
ID        PEER             TYPE    ID        ID        ID        ID        STATE    UPTIME           R/I/S  
-----------------------------------------------------------------------------------------------------------------
0         100.0.0.101      vsmart  1         1         100       None      up       13:13:06:55      6/6/4
```

Verify that **BFD (Bidirectional Forwarding Detection) sessions** are successfully established among all WAN-Edge routers in the topology. BFD sessions are critical for ensuring fast detection of link failures and maintaining the reliability of the network. Confirming the establishment of these sessions ensures that the WAN-Edge routers can quickly react to any connectivity issues, contributing to the overall stability and performance of the SD-WAN fabric.
``` { .ios, .no-copy}
Stockholm-Branch#show sdwan bfd sessions 
                                      SOURCE TLOC      REMOTE TLOC                                      DST PUBLIC                      DST PUBLIC         DETECT      TX                              
SYSTEM IP        SITE ID     STATE       COLOR            COLOR            SOURCE IP                                       IP                                              PORT        ENCAP  MULTIPLIER  INTERVAL(msec  UPTIME          TRANSITIONS   
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
10.1.1.2         20          up          biz-internet     biz-internet     172.16.1.10                                     172.16.1.20                                     12366       ipsec  7           1000           0:00:46:34      2             
10.0.0.1         101         up          biz-internet     biz-internet     172.16.1.10                                     172.16.1.101                                    12346       ipsec  7           1000           0:00:46:34      4             
10.0.0.2         102         up          biz-internet     biz-internet     172.16.1.10                                     172.16.1.102                                    12346       ipsec  7           1000           0:00:46:33      2             
10.1.1.2         20          up          mpls             mpls             172.16.2.10                                     172.16.2.20                                     12366       ipsec  7           1000           0:00:46:34      2             
10.0.0.1         101         up          mpls             mpls             172.16.2.10                                     172.16.2.101                                    12346       ipsec  7           1000           0:00:46:34      4             
10.0.0.2         102         up          mpls             mpls             172.16.2.10                                     172.16.2.102                                    12346       ipsec  7           1000           0:00:46:34      2             

Stockholm-Branch# 
```

!!! hint
    To quickly verify OMP peering across the SD-WAN fabric, log in to the SD-WAN Controller (vSmart) and confirm that OMP peering is successfully established with all WAN-Edge routers using show command **show omp peers**. This step ensures that the control plane is fully operational, allowing the exchange of routes and policies between the SD-WAN Controller (vSmart) and all WAN-Edge devices.

Having verified that the control connections are established and all interfaces are operational, we are now ready to proceed to the next phase of the lab. In this step, we will explore various scenarios of service insertion and route leaking, leveraging the existing topology.