# Route traffic from Sydney to Stockholm via local firewall in Sydney in different VRF

## Introduction

In this lab exercise, you will analyze the traffic flow between a user in the **<font color="Green">EMEA-Stockholm-Branch (site-10)</font>** and a user in the **<font color="Green">APAC-Sydney-Branch (site-20)</font>**, with the traffic routed through a **firewall** **<font color="Green">(London-Hub-Firewall)</font>** hosted at the 
**EMEA-London-Hub (site-101)**. The **London-Hub-Firewall**, accessible via the **London-Hub** WAN-Edge router in **<font color="#9AAFCB">VRF-1</font>**, plays a critical role in inspecting and securing the traffic as it traverses the network. This exercise will focus on understanding the configuration and verification of service chaining, 
centralized policies, and the interactions between network elements to ensure the intended traffic flow through the designated security device.

Here is a breakdown of the key components involved in the network path:

- **Source:** The traffic originates from a **Sydney-User** user in the **APAC-Sydney-Branch (site-20)**.
- **Destination:** The intended recipient is a **Stockholm-User** in the **EMEA-Stockholm-Branch (site-10)**.
- **Firewall:** All traffic passes through a **firewall (Sydney-FW)**, which is hosted locally at the **APAC-Sydney-Branch (site-20)** in a **<font color="green">VRF-2</font>**.
- **WAN Edge Router:** The **Sydney-Branch** WAN-Edge router, configured in ***<font color="#9AAFCB">VRF-1</font>***, facilitates the traffic's reachability to the firewall and subsequent routing towards the destination.

Ensure that each component is properly configured and verify the traffic flow is going through **London-FW**.

!!! note
    Through this lab, firewall is configured to inspect traffic automatically in **inspected mode**, ***<font color="red"> without requiring any additional configuration</font>***. This inspection ensures that only safe and authorized traffic flows through the network, enhancing security and protecting against potential threats.

## Intended Traffic Flow Diagram

The following diagram illustrates the **<font color="orange">flow of traffic within the network for this scenario</font>**. Traffic is initiated from the **Sydney-User** and is first redirected to the **Sydney-Firewall** in a **<font color="green">VRF-2</font>** for <font color="orange">**inspection**</font>. After the traffic undergoes inspection, it is then forwarded to the **Stockholm-User** in the **Stockholm Branch**. 

This scenario demonstrates how traffic is securely routed through the firewall for inspection before reaching its final destination, ensuring that security policies are applied effectively within the SD-WAN fabric.

<figure markdown>
  ![Scenario-2 Traffic Flow](./assets/Scenario-3.gif)
</figure>
