# IP Addressing Scheme for Lab Topology 

The following IP addressing scheme outlines the network structure utilized in this lab topology. Each segment of the 
network is assigned a specific IP range to ensure clear communication paths and avoid address conflicts. This scheme 
is designed to facilitate seamless routing, service insertion, and service chaining within the SD-WAN environment. 
Adhering to this predefined IP addressing plan ensures consistent configuration and simplifies troubleshooting 
throughout the lab exercises.


## Stockholm Branch

| System IP        | 10.1.1.1        |
|------------------|-----------------|
| GigabitEthernet3 | 192.168.10.1/24 |
| GigabitEthernet4 | 10.10.10.1/24   |
| Stockholm-FW     | 10.10.10.2/24   |
| Stockholm-User   | 192.168.10.2/24 |


## Sydney Branch

| System IP        | 10.1.1.2        |
|------------------|-----------------|
| GigabitEthernet3 | 192.168.20.1/24 |
| GigabitEthernet4 | 10.20.20.1/24   |
| Sydney-FW        | 10.20.20.2/24   |
| Sydney-User      | 192.168.20.2/24 |


## London Branch

| System IP        | 10.0.0.1        |
|------------------|-----------------|
| GigabitEthernet4 | 10.101.101.1/24 |
| London-FW        | 10.101.101.2/24 |

## Singapore Branch

| System IP        | 10.0.0.2        |
|------------------|-----------------|
| GigabitEthernet4 | 10.102.102.1/24 |
| Singapore-FW     | 10.102.102.2/24 |


