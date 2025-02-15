# Amsterdam-Branch WAN-Edge Router Unable to Join Fabric and Traffic Inspection Requirements

## Description:
The IT Administrator is in the process of bringing the **Amsterdam-Branch** online; however, the **Amsterdam-Branch** WAN-Edge router is facing issues and is unable to join the **SD-WAN fabric**. This is preventing the branch from establishing connectivity with other sites within the network and **Internet**.

Priority: **<font color="red">High</font>**

Total Faults: **<font color="green">6</font>**

### Topology Diagram


![Ticket Topology](./assets/Challenge-Topology.png){usemap="#image-map"}
<map name="image-map">
     <area target="_self" alt="Stockholm-Branch" title="Stockholm-Branch" href="telnet://127.0.0.1:9013" coords="232,611,28" shape="circle">
     <area target="_self" alt="Stockholm-FW" title="Stockholm-FW" href="telnet://127.0.0.1:9015" coords="37,594,70,618" shape="rect">
     <area target="_self" alt="Stockholm-User" title="Stockholm-User" href="telnet://127.0.0.1:9016" coords="35,678,74,707" shape="rect">
     <area target="_self" alt="Amsterdam-Branch" title="Amsterdam-Branch" href="telnet://127.0.0.1:9000" coords="406,665,25" shape="circle">
     <area target="_self" alt="Amsterdam-User" title="Amsterdam-User" href="telnet://127.0.0.1:9002" coords="587,649,623,673" shape="rect">
     <area target="_self" alt="Sydney-Branch" title="Sydney-Branch" href="telnet://127.0.0.1:9017" coords="783,606,23" shape="circle">
     <area target="_self" alt="Sydney-FW" title="Sydney-FW" href="telnet://127.0.0.1:9019" coords="948,583,982,610" shape="rect">
     <area target="_self" alt="Sydney-User" title="Sydney-User" href="telnet://127.0.0.1:9020" coords="949,677,983,700" shape="rect">
     <area target="_self" alt="London-Branch" title="London-Branch" href="telnet://127.0.0.1:9006" coords="293,206,29" shape="circle">
     <area target="_self" alt="London-FW" title="London-FW" href="telnet://127.0.0.1:9008" coords="101,191,135,216" shape="rect">
     <area target="_self" alt="Singapore-Branch" title="Singapore-Branch" href="telnet://127.0.0.1:9010" coords="552,206,27" shape="circle">
     <area target="_self" alt="Singapore-FW" title="Singapore-FW" href="telnet://127.0.0.1:9012" coords="714,186,753,212" shape="rect">
     <area target="_self" alt="Controller-1" title="Controller-1" href="telnet://127.0.0.1:9003" coords="429,29,30" shape="circle">
</map>


### Intent

The IT Administrator aims to achieve full connectivity between all sites, ensuring seamless communication across the SD-WAN fabric.
Additionally, any traffic destined for the **Internet** must be inspected by the **<font color="green">London-Firewall</font>** before being routed through the **London-Branch** WAN-Edge router to the **<font color="green">Internet</font>**.

### Action Required

- Investigate and resolve the issue preventing the **Amsterdam-Branch** WAN-Edge router from joining the fabric.
- Verify and ensure that the configuration aligns with the desired intent of achieving **<font color="green">site-to-site connectivity</font>**.
- Implement the necessary policies to ensure that all **Internet-bound** traffic is directed to the **London-Firewall** for inspection and subsequently routed to the Internet through the **London-Branch** WAN-Edge router.
- The IT Administrator has already **configured** the **<font color="green">Amsterdam-Branch policy</font>** on the SD-WAN Manager. **<font color="green">Activate the policy</font>** and verify that it is correctly configured to achieve the intended objectives.
### Intended Traffic Flow Diagram

The following diagram illustrates the **<font color="orange">flow of Internet traffic for the Amsterdam-Branch</font>**.

<figure markdown>
  ![Ticket Flow Topology](./assets/Challenge.gif)
</figure>

### Final verification for Troubleshooting-Pro

1. **<font color="green">Amsterdam-Branch</font>** centralized policy **<font color="green">activated</font>** in **SD-WAN Manager**.
2. Successful ping from **<font color="green">Amsterdam-User</font>** towards **<font color="orange">Sydney-User</font>** and **<font color="orange">Stockholm-User</font>**. 
3. Successful _traceroute_ from **Amsterdam-User** towards **<font color="green">Internet</font>** **8.8.8.8** via **London-FW**. 