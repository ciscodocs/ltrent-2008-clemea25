# IP Addressing Scheme for Lab Topology 

The following IP addressing scheme outlines the network structure utilized in this lab topology. Each segment of the 
network is assigned a specific IP range to ensure clear communication paths and avoid address conflicts. This scheme 
is designed to facilitate seamless routing, service insertion, and service chaining within the SD-WAN environment. 
Adhering to this predefined IP addressing plan ensures consistent configuration and simplifies troubleshooting 
throughout the lab exercises.

!!! info 
    * All **<font color="green">biz-internet</font>** TLOCs have IP address of <font color="green">172.16.1.**<site-id>**/24</font>
    * All **<font color="green">mpls</font>** TLOCs have IP address of <font color="green">172.16.2.**<site-id>**/24<font>

!!! info
    * All **<font color="green">GigabitEthernet 3</font>** interfaces have standard IP addressing schema **192.168.site-id.1/24**.
    * All **<font color="green">GigabitEthernet 4</font>** interfaces have standard IP addressing schema **10.site-id.site-id.1/24**.


## Stockholm Branch (Site-ID 10)

| System IP        | 10.1.1.1        |
|------------------|-----------------|
| GigabitEthernet3 | 192.168.10.1/24 |
| GigabitEthernet4 | 10.10.10.1/24   |
| Stockholm-FW     | 10.10.10.2/24   |
| Stockholm-User   | 192.168.10.2/24 |

## Sydney Branch (Site-ID 20)

| System IP        | 10.1.1.2        |
|------------------|-----------------|
| GigabitEthernet3 | 192.168.20.1/24 |
| GigabitEthernet4 | 10.20.20.1/24   |
| Sydney-FW        | 10.20.20.2/24   |
| Sydney-User      | 192.168.20.2/24 |


## London Branch (Site-ID 101)

| System IP        | 10.0.0.1        |
|------------------|-----------------|
| GigabitEthernet4 | 10.101.101.1/24 |
| London-FW        | 10.101.101.2/24 |

## Singapore Branch (Site-ID 102)

| System IP        | 10.0.0.2        |
|------------------|-----------------|
| GigabitEthernet4 | 10.102.102.1/24 |
| Singapore-FW     | 10.102.102.2/24 |


