- [STP](#stp)
  - [Fundamentals](#fundamentals)
    - [versions](#versions)
  - [802.1D STP](#8021d-stp)
    - [port types](#port-types)
    - [Terminology](#terminology)
    - [Spanning Tree Path Cost](#spanning-tree-path-cost)
    - [Building STP topology](#building-stp-topology)
      - [root bridge election](#root-bridge-election)

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

### Building STP topology

#### root bridge election

* If the neighbor’s configuration BPDU is preferred to its own BPDU, the switch updates its BPDUs to include the new root bridge identifier along with a new root path cost that correlates to the total path cost to reach the new root bridge. This process continues until all switches in a topology have identified the root bridge switch.
* more preferable if the **priority** in the bridge identifier is **lower**