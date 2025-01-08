# Lab 5 - Getting Started

# Routing Traffic Through Firewall in Singapore Branch Across VRFs for Site-to-Site Communication

## Introduction
In this lab exercise, you will explore the traffic flow from a user located in the **APAC-Sydney-Branch (site-20)** breakout directly to **<font color="green">Internet</font>** via underlay. Here is a breakdown of the key components involved in the network path: 

Here is a breakdown of the key components involved in the network path:

- **Source:** The traffic originates from a **Sydney-User** in the **APAC-Sydney-Branch (site-20)**.
- **Destination:** Internet for example **<font color="orange">Google DNS server 8.8.8.8</font>**.
- **Firewall:** All traffic passes through a **firewall (Sydney-FW)**, which is hosted locally at the **APAC-Sydney-Branch (site-20)** in **<font color="green">VRF-2</font>**.
- **WAN Edge Router:** The **Sydney-Branch** WAN-Edge router, router carries user traffic in ***<font color="#9AAFCB">VRF-1</font>***  and facilitates the firewall reachability in ***<font color="#9AAFCB">VRF-2</font>***. 

Ensure that each component is properly configured and verify the traffic flow is going through **Sydney-FW**.

!!! note
    Through this lab, firewall is configured to inspect traffic automatically in **inspected mode**, ***<font color="red"> without requiring any additional configuration</font>***. This inspection ensures that only safe and authorized traffic flows through the network, enhancing security and protecting against potential threats.

## Intended Traffic Flow Diagram

The following diagram illustrates the **<font color="orange">flow of traffic within the network for this scenario</font>**. Traffic is initiated from the **Stockholm-User** and is first redirected to the **Singapore-Firewall** in a **<font color="green">VRF-2</font>** for <font color="orange">**inspection**</font>. After the traffic undergoes inspection, it is then forwarded to the **Sydney-User** in the **Sydney Branch**. 

This scenario demonstrates how traffic is securely routed through the firewall for inspection before reaching its final destination, ensuring that security policies are applied effectively within the SD-WAN fabric.

<figure markdown>
  ![Scenario-2 Traffic Flow](./assets/Scenario-5.gif)
</figure>