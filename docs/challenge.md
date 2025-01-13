# Amsterdam-Branch WAN-Edge Router Unable to Join Fabric and Traffic Inspection Requirements

## Description:
The IT Administrator is in the process of bringing the **Amsterdam-Branch** online; however, the **Amsterdam-Branch** WAN-Edge router is facing issues and is unable to join the **SD-WAN fabric**. This is preventing the branch from establishing connectivity with other sites within the network.

Priority: **High**

### Topology Diagram

<figure markdown>
  ![Ticket Topology](./assets/Challenge-Topology.png)
</figure>

### Intent

The IT Administrator aims to achieve full connectivity between all sites, ensuring seamless communication across the SD-WAN fabric.
Additionally, any traffic destined for the **Internet** must be inspected by the **<font color="green">London-Firewall</font>** before being routed through the **London-Branch** WAN-Edge router to the **<font color="green">Internet</font>**.

### Action Required

- Investigate and resolve the issue preventing the **Amsterdam-Branch** WAN-Edge router from joining the fabric.
- Verify and ensure that the configuration aligns with the desired intent of achieving **<font color="green">site-to-site connectivity</font>**.
- Implement the necessary policies to ensure that all **Internet-bound** traffic is directed to the **London-Firewall** for inspection and subsequently routed to the Internet through the **London-Branch** WAN-Edge router.



### Intended Traffic Flow Diagram

The following diagram illustrates the **<font color="orange">flow of traffic within the network for the Amsterdam-Branch</font>**.

<figure markdown>
  ![Ticket Flow Topology](./assets/Challenge.gif)
</figure>