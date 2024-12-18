# Lab 1 - Routing Stockholm to Sydney Traffic Through Local Firewall in Same VRF-1 Network

## Introduction

In this lab exercise, you will explore the traffic flow from a user located in the **EMEA-Stockholm-Branch (site-10)** to a user situated at the **APAC-Sydney-Branch (site-20)**. 

Here is a breakdown of the key components involved in the network path:

- **Source:** The traffic originates from a **Stockholm-User** user in the **EMEA-Stockholm-Branch (site-10)**.
- **Destination:** The intended recipient is a **Sydney-User** in the **APAC-Sydney-Branch (site-20)**.
- **Firewall:** All traffic passes through a **firewall (Stockholm-FW)**, which is hosted locally at the **EMEA-Stockholm-Branch (site-10)**.
- **WAN Edge Router:** The **Stockholm-Branch** WAN-Edge router, configured in ***<font color="blue">VRF-1</font>***, facilitates the traffic's reachability to the firewall and subsequent routing towards the destination.

Ensure that each component is properly configured and verify the traffic flow is going through **Stockholm-FW**.

!!! note
    Through this lab, firewall is configured to inspect traffic automatically in **inspected mode**, ***<font color="red"> without requiring any additional configuration</font>***. This inspection ensures that only safe and authorized traffic flows through the network, enhancing security and protecting against potential threats. 

Please use the following credentials to connect to device:

| <!-- -->         | <!-- -->         |
| ---------------- | ---------------- |
| `IP Address`     | 1.1.1.1          |
| `Username`       | admin            |
| `Password`       | C1sco123         |

My content

Cisco IOS code block:

```ios
hostname ABC
interface GigabitEthernet1
 ip address 122.1.1.1
```

## Traffic Flow Diagram 

<figure markdown>
  ![Scenario-1](./assets/Scenario-1.gif)
</figure>

