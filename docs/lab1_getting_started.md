# Lab 1 - Route traffic from Stockholm to Sydney via local firewall in Stockholm

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

## Traffic Flow Diagram 

The following diagram illustrates the **<font color="orange">flow of traffic within the network for this scenario</font>**. Traffic is initiated from the **Stockholm-User** and is first redirected to the **Stockholm-Firewall** for <font color="orange">**inspection**</font>. After the traffic undergoes inspection, it is then forwarded to the **Sydney-User** in the **Sydney Branch**. This scenario demonstrates how traffic is securely routed through the firewall for inspection before reaching its final destination, ensuring that security policies are applied effectively within the SD-WAN fabric.
<figure markdown>
  ![Scenario-1 Traffic Flow](./assets/Scenario-1.gif)
</figure>

