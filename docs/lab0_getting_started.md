# Lab Topology Overview and Connectivity Verification

## Understanding the Lab Topology
This lab features a topology comprising four WAN-Edge routers strategically deployed across two regions: EMEA and APAC. In the EMEA region, the routers are located in London and Stockholm, while in the APAC region, they are deployed in Singapore and Sydney. Each WAN-Edge router is integrated with a firewall, which is essential for redirecting traffic for inspection across various use cases. The deployment leverages two transport types Internet and MPLS to provide reliable connectivity between the WAN-Edge routers across regions. This setup creates a robust environment for simulating real-world traffic routing and inspection scenarios.

## Device Connectivity and Interfaces

In the lab topology, each WAN-Edge router is configured with dual transport connectivity: the **GigabitEthernet 1** interface is connected to the **Internet transport**, while the **GigabitEthernet 2** interface is connected to the **MPLS transport**. For simplification, the deployment includes a single SD-WAN validator (vBond) and a single SD-WAN controller (vSmart), each assigned the following system ip mentioned in below table.

| <font color="blue">Device</font> | <font color="blue">System IP</font>   |
|--------------------------------|-------------|
| Validator  (vBond)             | 100.0.0.201 |
| Controller (vSmart)            | 100.0.0.101 |
| Manager    (vManage)           | 100.0.0.1   |

Additionally, the **GigabitEthernet 4** interface on each region WAN-Edge router is connected to a **firewall** within the service VPN. ***Branch users*** are also part of the service VPN and are connected to the WAN-Edge routers through the **GigabitEthernet 3** interface, as shown in the topology diagram. This setup enables seamless traffic flow and inspection while ensuring a realistic representation of enterprise network environments.

## Topology diagram

<figure markdown>
  ![topo](./assets/sdwan-topology.png)
</figure>

## Lab Connectivity

## Verifying Lab Connectivity

``` { .ios, .no-copy }
London-Hub#show sdwan control local-properties 
personality                       vedge
sp-organization-name              cml-sdwan-lab-tool
organization-name                 cml-sdwan-lab-tool
root-ca-chain-status              Installed
root-ca-crl-status                Not-Installed

certificate-status                Installed
certificate-validity              Valid
certificate-not-valid-before      Nov 26 15:34:51 2024 GMT
certificate-not-valid-after       Nov 24 15:34:51 2034 GMT

enterprise-cert-status            Not Applicable
enterprise-cert-validity          Not Applicable
enterprise-cert-not-valid-before  Not Applicable
enterprise-cert-not-valid-after   Not Applicable

dns-name                          validator.sdwan.local
site-id                           101
domain-id                         1
protocol                          dtls
tls-port                          0
system-ip                         10.0.0.1
chassis-num/unique-id             C8K-27148AFE-7160-69D3-ACC5-F77560530D63
serial-num                        2960F4DD
subject-serial-num                N/A
enterprise-serial-num             No certificate installed
token                             Invalid
keygen-interval                   1:00:00:00
retry-interval                    0:00:00:18
no-activity-exp-interval          0:00:00:20
dns-cache-ttl                     0:00:00:00
port-hopped                       FALSE
time-since-last-port-hop          0:00:00:00
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
GigabitEthernet1              172.16.1.101    12346  172.16.1.101    ::                                      12346    1/1  biz-internet     up     2     yes/yes/no   No/No  0:00:00:02   0:06:17:23  N    5  Default N/A                           
GigabitEthernet2              172.16.2.101    12346  172.16.2.101    ::                                      12346    1/0  mpls             up     2     yes/yes/no   No/No  0:00:00:02   0:06:17:23  N    5  Default N/A                           
```

