- [STP](#stp)
  - [Fundamentals](#fundamentals)
    - [versions](#versions)
  - [802.1D STP](#8021d-stp)
    - [port types](#port-types)
    - [Terminology](#terminology)
    - [Spanning Tree Path Cost](#spanning-tree-path-cost)
    - [Building STP topology](#building-stp-topology)
      - [root bridge election](#root-bridge-election)
      - [identifying root ports](#identifying-root-ports)
      - [identify block ports](#identify-block-ports)

# STP

* Problem: redundant links between switches adds reliability, but also causing possible loops
  * broadcasted ARP traffic may well trigger the loop
  * no TTL in layer 2

## Fundamentals

* idea: prevent loop by temporarily blocking traffic on redundant ports
* protocol: BPDU (bridge protocol data units)
  * enable switches to become aware of other switches
* steps: 
  * select a specific switch as the best / root switch

### versions

STP has multiple iterations:

* 802.1D, which is the original specification
* Per-VLAN Spanning Tree (PVST)
* Per-VLAN Spanning Tree Plus (PVST+)
* 802.1W Rapid Spanning Tree Protocol (RSTP)
* 802.1S Multiple Spanning Tree Protocol (MST)

## 802.1D STP

### port types

* Root port (RP)
  * only one RP per VLAN
* Designated port (DP)
  * recv/sent BPDU
  * only one active DP per link
* Blocking port
  * not forwarding traffic

### Terminology

* **Root bridge**
  * top / root of the tree
  * all ports are DP
* **BPDU**
  * dest. MAC addr: 01:80:c2:00:00:00
  * build the tree and notify the changes
  * two types:
    * Config BPDU
    * Topology change notification (TCN) BDPU
* **Root path cost**: combined cost for a specific path toward the root switch
* **System priority**: This 4-bit value indicates the preference for a switch to be root bridge. 
  * default: 32,768.
* **System ID extension**: This 12-bit value indicates the VLAN that the BPDU cor- relates to.
* **bridge identifier**: This is a combination of **MAC address, system ID extension, and system priority** of the bridge.
  * root bridge id
  * local bridge id
* **Max age**: If a switch loses contact with the BPDU’s source, it assumes that the BPDU information is still valid for the duration of the Max Age timer.
  * default: 20 seconds
  * config: `spanning-tree vlan vlan-id max-age maxage`. 
* **Hello time**: time interval that a BPDU is advertised out of a port. 
  * default: 2 seconds
  * config: `spanning-tree vlan vlan-id hello-time hello-time`.
* **Forward delay**: This is the amount of time that a port stays in a listening and learning state. 
  * default: 15 seconds
  * config: `spanning-tree vlan vlan-id forward-time forward-time`.
* **note**: STP was defined before modern switches existed. The devices that originally used STP were known as bridges. The terms *bridge* and *switch* are interchangeable in this context.

### Spanning Tree Path Cost

* **short mode**: 16-bit value, with a reference value of 20 Gbps.
* **long mode**: 32-bit value, with a reference speed of 20 Tbps.

![](img/2024-09-19-09-41-37.png)

### Building STP topology

#### root bridge election

* If the neighbor’s configuration BPDU is preferred to its own BPDU, the switch updates its BPDUs to include the new root bridge identifier along with a new root path cost that correlates to the total path cost to reach the new root bridge. This process continues until all switches in a topology have identified the root bridge switch.
* more preferable if the **priority** in the bridge identifier is **lower**

![](img/2024-09-19-09-38-41.png)

* The priority in the configuration BPDU packets is actually the priority plus the value of the sys-id-ext (which is the VLAN number).
  * looking at VLAN 10, which has a priority of 32,778, which is 10 higher than 32,768.

#### identifying root ports

1. The interface associated to lowest path cost is more preferred.
2. The interface associated to the lowest system priority of the advertising switch is preferred next.
3. The interface associated to the lowest system MAC address of the advertising switch is preferred next.
4. When multiple links are associated to the same switch, the lowest port priority from the advertising switch is preferred.
5. When multiple links are associated to the same switch, the lower port number from the advertising switch is preferred.

![](img/2024-09-19-09-50-09.png)
![](img/2024-09-19-09-50-39.png)

#### identify block ports

1. The interface is a designated port and must not be considered an RP.
2. The switch with the lower path cost to the root bridge forwards packets, and the one with the higher path cost blocks. If they tie, they move on to the next step.
3. The system priority of the local switch is compared to the system priority of the remote switch. The local port is moved to a blocking state if the remote system prior- ity is lower than that of the local switch. If they tie, they move on to the next step.
4. The system MAC address of the local switch is compared to the system MAC address of the remote switch. The local designated port is moved to a blocking state if the remote system MAC address is lower than that of the local switch.