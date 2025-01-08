# Lab 5 - Direct Internet Breakout for Sydney-Branch Traffic via Underlay with Firewall Inspection

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

The following diagram illustrates the **<font color="orange">flow of traffic within the network for this scenario</font>**. Traffic is initiated from the **Sydney-User** and is first redirected to the **Sydney-Firewall** in a **<font color="green">VRF-2</font>** for <font color="orange">**inspection**</font>. After the traffic undergoes inspection, it is then forwarded to the **Internet( for example Google <font color="green">DNS server 8.8.8.8</font>)**. 

This scenario demonstrates how traffic is securely routed through the firewall for inspection before reaching its final destination, ensuring that security policies are applied effectively within the SD-WAN fabric.

<figure markdown>
  ![Scenario-2 Traffic Flow](./assets/Scenario-5.gif)
</figure>

## Traffic flow without any policy

In the initial configuration, prior to applying any traffic policies or making modifications to the configuration group, it is observed that connectivity to the Internet, specifically to the **<font color="orange">Google DNS server 8.8.8.8</font>**, is not established from **VRF-1** on the **Sydney-Branch** WAN-Edge router. This lack of connectivity indicates that the current setup does not support direct Internet access from **VRF-1**, highlighting the need for further configuration, such as route adjustments, NAT settings, or centralized policies, to enable Internet reachability. This baseline observation provides a starting point for implementing the necessary changes in subsequent steps.

```{ .ios .no-copy }
Sydney-Branch#show ip route vrf 1

Routing Table: 1
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, m - OMP
       n - NAT, Ni - NAT inside, No - NAT outside, Nd - NAT DIA
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       H - NHRP, G - NHRP registered, g - NHRP registration summary
       o - ODR, P - periodic downloaded static route, l - LISP
       a - application route
       + - replicated route, % - next hop override, p - overrides from PfR
       & - replicated local route overrides by connected

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
m        10.10.10.0/24 [251/0] via 10.1.1.1, 1d10h, Sdwan-system-intf
m        10.101.101.0/24 [251/0] via 10.0.0.1, 2d01h, Sdwan-system-intf
m        10.102.102.102/32 [251/0] via 10.0.0.2, 2d01h, Sdwan-system-intf
m     192.168.10.0/24 [251/0] via 10.1.1.1, 1d10h, Sdwan-system-intf
      192.168.20.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.20.0/24 is directly connected, GigabitEthernet3
L        192.168.20.1/32 is directly connected, GigabitEthernet3
Sydney-Branch#
```
The **Sydney-Branch** WAN-Edge router is able to reach the **Internet** through the **<font color="green">underlay network</font>**, confirming that basic Internet connectivity is functional at the transport level. 

```{.ios .no-copy}
Sydney-Branch#ping 8.8.8.8
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 8.8.8.8, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 8/9/11 ms

Sydney-Branch#ping 8.8.4.4
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 8.8.4.4, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 7/8/10 ms
Sydney-Branch#
```
However, Internet access from **<font color="orange">VRF-1</font>** is not established, as demonstrated in the output below. This indicates that while the underlay network is properly configured for external reachability, additional configurations, such as **NAT**, are required to enable Internet connectivity for **<font color="green">VRF-1</font>**.

```{.ios .no-copy}
Sydney-Branch#ping vrf 1 8.8.8.8
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 8.8.8.8, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)

Sydney-Branch#ping vrf 1 8.8.4.4
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 8.8.4.4, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
Sydney-Branch#
```
## Enabling NAT in Configuration Group for Sydney-Branch 

In Cisco Catalyst SD-WAN, enabling **NAT** (Network Address Translation) on the interface is crucial for reaching the underlay network for **Direct Internet Access (DIA)**. NAT translates internal private IP addresses into public IP addresses, allowing devices in **private VRFs** to communicate with external networks, such as the **Internet**. Without NAT, traffic from the private VRF cannot traverse the underlay network to access external resources. To enable NAT on the **<font color="green">Sydney-Branch</font>** WAN-Edge router, follow these steps:
