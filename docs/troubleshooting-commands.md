# Troubleshooting commands

The following troubleshooting commands on WAN-Edge router are essential tools for diagnosing and resolving issues related to service 
insertion and service chaining in an SD-WAN environment. These commands provide detailed insights into the configuration, 
operational status, and flow of traffic through service nodes, such as firewalls or other network services. 

By employing these commands, administrators/engineers/architects can efficiently identify and address misconfigurations, 
connectivity issues, or policy-related discrepancies. Additionally, documenting these commands as part of your operational toolkit ensures 
they are readily available for future debugging scenarios, streamlining the troubleshooting process and maintaining 
network performance and reliability.

## Show running commands

- **show sdwan running-config sdwan**

## Control connections show commands

- **show sdwan control local-properties**
- **show sdwan control connections**

## OMP peering and routes show commands

- **show sdwan omp peers**
- **show sdwan omp routes**
- **show sdwan omp services**

## Service Chaining/Insertion show commands

- **show platform software sdwan service-chain database**
- **show platform software sdwan service-chain stats detail**
- **show platform hardware qfp active feature sdwan datapath service-chain database**
- **show platform hardware qfp active feature sdwan datapath service-chain service-chain stats**

## ACL show commands

- **show sdwan policy data-policy-filter**
- **show sdwan policy access-list-names**
- **show sdwan policy access-list-associations**
- **show sdwan policy access-list-counters**

