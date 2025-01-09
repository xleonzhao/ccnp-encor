# protocol numbers

* EIGRP: 88
* OSPF: 89
* IGMPv2: 2
* PIM: 103
* IPSEC AH: 51
* IPSEC ESP: 50

# ch. 1-5: total 47, wrong: 2, arguable: 1

* https://itexamanswers.net/ccnp-encor-v8-chapters-1-5-l2-redundancy-test-online.html
* ? 16: Why is it important that the network administrator consider the spanning-tree network diameter when choosing the root bridge?
  * > BPDUs may be discarded because of expiring timers.
  * The network diameter limitation is 9.
  * The cabling distance between the switches is 100 meters.
  * Convergence is slower as the BPDU travels away from the root.

> Explanation: The optional diameter keyword in the `spanning-tree vlan <vlan-id> root {primary | secondary} [diameter <diameter>]` command allows for tuning of the STP convergence. The diameter keyword should reference the maximum number of Layer 2 hops that a switch can be from the root bridge and modify the timers accordingly. The timers do not need to be modified on other switches because they are carried throughout the topology through the root bridge BPDUs.

* 30: Given the following configuration, which two statements are true? (Choose two.)
    ```
    switch(vlan)# vtp version 2
    switch(vlan)# vtp mode server
    switch(vlan)# vtp domain Cisco
    switch(vlan)# vtp password mypassword
    ```
  * This switch can advertise its VLAN configuration to other switches in the Cisco domain only, but can receive advertisements from other domains.
  * The password will prevent unauthorized routers from participating in the Cisco domain.
  * > This switch can send and receive advertisements from only the Cisco domain.
  * > This switch can create, modify, and delete all VLANs within the Cisco domain.
  * This switch maintains a full list of all VLANs and can create VLANs, but cannot delete or modify existing VLANs.

> Explanation: A switch in VTP server mode can create, modify, and delete VLANs as well as transmit that information (if the switch has the highest VTP configuration revision number) to other switches in the same VTP domain.

* 46: Which statement describes the term root guard in the operation of STP?
  * It is a feature that prevents any alternative or root ports from becoming designated ports because of a loss of BPDUs on the root port.
  * > It is a feature that prevents a configured port from becoming a root port.
  * It is a value that decides which switch can become the root bridge.
  * It is a safety mechanism that shuts down ports configured with STP portfast upon receipt of a BPDU.

# ch. 6: 26/34

* https://itexamanswers.net/ccnp-encor-v8-chapters-6-7-routing-essentials-and-eigrp-test-online.html

* 2: What are two characteristics of link-state routing protocols? (Choose two.)
  * They can load balance across unequal metric cost paths.
  * They use path attributes to determine the best loop-free path.
  * They periodically send full routing table updates to directly connected neighbors.
    * this is incorrect, during periodical refreshing, only its originated LSAs are re-advertised. ([see here](./06-routing-basics.md#periodical-refreshing))
  * > They provide routers with a synchronized identical map of the network.
  * > They use more CPU and memory resources than distance vector protocols do.

* 15: Refer to a portion of an EIGRP topology table:
```
P 172.18.3.0/24, 1 successors, __ is 2172416
via 172.18.4.3 (2172416/28160), Serial0/0/1
P 172.18.5.0/24, 1 successors, __ is 2495120
via 172.18.6.3 (2495120/227692), Serial0/1/0
P 192.168.24.0/24, 1 successors, __ is 2684416
via 192.168.13.5 (2684416/2072316), Serial0/0/0
via 172.18.4.3 (2854912/2342912), Serial0/0/1
P 10.34.1.0/24, 1 successors, __ is 3072
via 10.13.1.3 (3072/2937), GigabitEthernet0/1
via 10.14.1.4 (5376/2937), GigabitEthernet0/2
```
What is the reported distance of the successor route for 172.18.3.0/24?
  * 2684416
  * 2172416
  * 2072316
  * > 28160

* 20: Which statement describes the autonomous system number used in EIGRP configuration on a Cisco router?
  * It carries the geographical information of the organization.
  * It is a globally unique autonomous system number that is assigned by IANA.
  * > It functions as a process ID in the operation of the router.
  * It identifies the ISP that provides the connection to network of the organization.

* 22: Refer to the exhibit. All the routers that are displayed are part of the EIGRP domain. Assuming EIGRP metric weights are not altered in the configurations, which path will a packet take that originates from a host on the 192.168.1.0/24 network and is going to a host on the 192.168.2.0/24 network?
  * R1, R2, R3
  * R1, R5, R3
  * R1, R4, R3
  * > R1, R2, R5, R3

![](img/2025-01-08-20-49-29.png)

> EIGRP only uses the slowest bandwidth in its composite metric. The slowest bandwidth in the path R1, R2, R5, R3 is 64 kb/s and thus offers the best path to the 192.168.2.0/24 network.

? didn't get it

* 23: Which two metric weights are set to one by default when costs in EIGRP are being calculated? (Choose two.)
  * k6
  * > k3
  * > k1
  * k4
  * k5
  * k2

* 26: If all router Ethernet interfaces in an EIGRP network are configured with the default EIGRP timers, how long will a router wait by default to receive an EIGRP packet from its neighbor before declaring the neighbor unreachable?
  * 10 seconds
  * > 15 seconds
  * 20 seconds
  * 30 seconds

* 34: What is the administrative distance of an EIGRP summary route?
  * 110
  * > 5
  * 90
  * 20