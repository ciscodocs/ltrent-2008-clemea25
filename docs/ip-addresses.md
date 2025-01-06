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
| Stockholm-FW     | 10.20.20.2/24   |
| Stockholm-User   | 192.168.20.2/24 |


