- [OSI model](#osi-model)
- [layer 2 forwarding](#layer-2-forwarding)
  - [MAC Table](#mac-table)
    - [manually change the mac-table](#manually-change-the-mac-table)
    - [Timer for mac address table entries](#timer-for-mac-address-table-entries)
  - [VLAN](#vlan)
    - [Access Ports](#access-ports)
    - [Trunk Ports](#trunk-ports)
    - [Native VLAN](#native-vlan)
    - [VLAN switching](#vlan-switching)
  - [Troubleshooting](#troubleshooting)
    - [check mac table](#check-mac-table)
    - [check interface/port status](#check-interfaceport-status)
  - [Lab](#lab)
- [Layer-3 forwarding](#layer-3-forwarding)
  - [Local Network Forwarding](#local-network-forwarding)
  - [Packet Routing](#packet-routing)
  - [IP Address Assignment](#ip-address-assignment)
    - [Routed Subinterfaces](#routed-subinterfaces)
    - [Switched Virtual Interfaces (SVI)](#switched-virtual-interfaces-svi)
    - [Routed Switch Ports](#routed-switch-ports)
  - [Verification of IP Addresses](#verification-of-ip-addresses)
- [Forwarding Architecture](#forwarding-architecture)
  - [Process switching](#process-switching)
    - [Data structures](#data-structures)
  - [CEF](#cef)
    - [TCAM](#tcam)
    - [Distributed Forwarding](#distributed-forwarding)
    - [Software CEF](#software-cef)
    - [Hardware CEF](#hardware-cef)
    - [SDM templates](#sdm-templates)

# OSI model

| Layer   | Name         | Description                                       |
|---------|--------------|---------------------------------------------------|
| Layer 7 | Application  | Interface for receiving and sending data          |
| Layer 6 | Presentation | Formatting of data and encryption                 |
| Layer 5 | Session      | **Tracking** of packets                           |
| Layer 4 | Transport    | **End-to-end** communication between devices      |
| Layer 3 | Network      | Logical **addressing** and **routing** of packets |
| Layer 2 | Data Link    | Hardware **addressing**                           |
| Layer 1 | Physical     | Media type and connector                          |

* Nice way to describe data link layer and network layer with a focus on addressing

# layer 2 forwarding

## MAC Table

* switch maintains a MAC table
  * MAC: Media Access Control
  * The MAC address table resides in content addressable memory (CAM).
    * basically a hash table alike thing

| MAC address        | port                       |
|--------------------|----------------------------|
| PC A's mac address | port N where A connects to |

* in case there is an unknown MAC address, switch will do broadcast to every port, called `unknown unicast flooding`
* src MAC addr/dst MAC addr will not be changed by switches/bridges/hubs

```
show mac address-table [address mac-address | dynamic | vlan vlan-id]
```

* `dynamic` means the entry is learned by switch, not statically configured

### manually change the mac-table

* Some older technologies (such as load balancing) require a static MAC address entry in the MAC address table to prevent unknown unicast flooding. 

```
mac address-table static mac-address vlan vlan-id {drop | interface interface-id}
clear mac address-table dynamic [{address mac-address | interface interface-id | vlan vlan-id}]
```

### Timer for mac address table entries

* _aging-time_
  * default: 300s
  * change it: `mac address-table aging-time <seconds>`

## VLAN

* 802.1Q
* each vlan is a broadcast domain
  * uniquely identified by VLAN ID
  * may across multiple switches
  * mac table will be per VLAN
* VLAN header
  * 16bits after MAC address in ethernet frame

![](img/vlan.png)

```
SW1# configure term
Enter configuration commands, one per line. End with CNTL/Z.
SW1(config)# vlan 10
SW1(config-vlan)# name PCs
SW1(config-vlan)# vlan 20
SW1(config-vlan)# name Phones

show vlan [{brief | id vlan-id | name vlan-name | summary}]
```

### Access Ports

* An Access ports is assigned to only one VLAN.
  * The 802.1Q tags are not included on packets transmitted or received on access ports.

```
SW1(config-vlan)# interface gi1/0/15
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 99
```

* Catalyst access switches place switch ports as Layer 2 access ports for VLAN 1 by default.

---

* `swtichport host` is a shortcut to put port in access and enable portfast

### Trunk Ports

* Trunk ports can **carry** multiple VLANs.

```
SW1(config)# interface gi1/0/2
SW1(config-if)# switchport mode trunk
```

* load balancing between network links where select VLANs are allowed on one
trunk link, while a different set of VLANs are allowed on a different trunk port.

```
switchport trunk allowed vlan {vlan-ids | all | none | add vlan-ids | remove vlan-ids | except vlan-ids}
! e.g.:
switchport trunk allowed vlan 1,10,20,99
```

* When you are scripting configuration changes, it is best to use the `add` and `remove` keywords because they are more prescriptive.

### Native VLAN

* On a trunk link, any untagged frames received are assumed to belong to the native VLAN
* All switch control plane /management traffic is advertised using native VLAN 1:
  * Cisco Discovery Protocol (CDP)
  * Link Layer Discovery Protocol (LLDP)
  * Spanning Tree Protocol (STP)
  * VTP (VLAN Trunking Protocol)
  * PAgP/LACP (for EtherChannel negotiation)
* The default native VLAN is VLAN 1
  * the default can be changed by `switchport trunk native vlan vlan-id`
* The native VLAN should match on both ports for traffic to be transmitted for that VLAN across the trunk link.
* The Cisco security hardening guidelines recommend changing the native VLAN to something other than VLAN 1.
  * it should be set to a VLAN that is not used at all.
  * everything should be tagged properly when entering the network

### VLAN switching

* based on **per-VLAN mac table** against _dest mac address_
  * recall mac addr are learned dynamically
```
SW1# show mac address-table dynamic
Mac Address Table
Vlan Mac Address    Type    Ports
------------------------------------
1    0081.c4ff.8b01 DYNAMIC Gi1/0/2
1    189c.5d11.9981 DYNAMIC Gi1/0/3
1    189c.5d11.99c7 DYNAMIC Gi1/0/3
10   5067.ae2f.6480 DYNAMIC Gi1/0/7
10   7069.5ad4.c220 DYNAMIC Gi1/0/13
10   e8ed.f3aa.7b98 DYNAMIC Gi1/0/12
20   189c.5d11.9981 DYNAMIC Gi1/0/3
20   7069.5ad4.c221 DYNAMIC Gi1/0/14
```

* endpoint -> access port -> trunk port
  * incoming frames are tagged internally to associate them with the configured VLAN
    * unless the frame belongs to native VLAN
  * check _dst mac addr_ against _VLAN mac table_ to find the outgoing port
    * if no mac addr found in mac table, do ARP (broadcasting) within VLAN
  * add VLAN headers when frames are sent out of the trunk port
* trunk -> trunk
  * VLAN headers are typically retained when frames are received / sent out of a trunk port
* trunk -> access -> endpoint
  * incoming frame still cariies the VLAN header
  * switch check the mac table and find the outgoing port is an access port
  * remove VLAN headers when frames are sent out of the access port to endpoints

## Troubleshooting

### check mac table

```
SW1# show mac address-table dynamic
```
* If multiple MAC addresses appear on the same port, you know that a switch, hub, or server with a virtual switch is connected to that switch port.

### check interface/port status

```
show interfaces gi1/0/5 switchport
show interfaces status
```

## Lab

* [vlan and trunk](./lab.md)

# Layer-3 forwarding

## Local Network Forwarding

* ARP: IP addr -> MAC addr
```
show ip arp [mac-address | ip-address | vlan vlan-id | interface-id]
```
* the destination MAC address is needed for the next-hop IP address.

## Packet Routing

* routing table
* routes
  * static configured
  * default gateway
  * learned via routing protocols
* ARP for next-hop IP

## IP Address Assignment

* `ip address <ip-address> <subnet-mask> [secondary]`
* interface IP/subnet goes to RIB with AD=0
* A `routed interface` is basically any interface on a router

### Routed Subinterfaces

* a single physical routed interface multiplexed for multiple VLANs
  * create a trunk port on the switch
  * create a logical subinterface on the router
    * which terminates VLAN at L3

```
R2# configure terminal
Enter configuration commands, one per line. End with CNTL/Z.
R2(config-if)# int g0/0/1.10
R2(config-subif)# encapsulation dot1Q 10
R2(config-subif)# ip address 10.10.10.2 255.255.255.0
R2(config-subif)# ipv6 address 2001:db8:10::2/64
R2(config-subif)# int g0/0/1.99
R2(config-subif)# encapsulation dot1Q 99
R2(config-subif)# ip address 10.20.20.2 255.255.255.0
R2(config-subif)# ipv6 address 2001:db8:20::2/64
```

### Switched Virtual Interfaces (SVI)

* aka _VLAN interface_
* assign an IP address to a switched virtual interface (SVI)/VLAN interface
* `interface vlan <vlan-id>`
* the SVIs can be used for **routing packets between VLANs** without the need of an external router
  * For simplicity and efficient design, use only one switch as the routing point

```
SW1(config)# interface vlan 10
SW1(config-if)# ip address 10.10.10.1 255.255.255.0
SW1(config-if)# ipv6 address 2001:db8:10::1/64
SW1(config-if)# no shutdown
SW1(config-if)# interface vlan 99
SW1(config-if)# ip address 10.99.99.1 255.255.255.0
SW1(config-if)# ipv6 address 2001:db8:99::1/64
SW1(config-if)# no shutdown
```

### Routed Switch Ports

* must be multilayer switch
* a port can be converted from a Layer 2 switch port to a routed switch port
  * `no switchport`
* e.g.: for building a point-to-point L3 link

```
SW1(config)# int gi1/0/14
SW1(config-if)# no switchport
SW1(config-if)# ip address 10.20.20.1 255.255.255.0
SW1(config-if)# ipv6 address 2001:db8:20::1/64
SW1(config-if)# no shutdown
```

## Verification of IP Addresses

* `show ip interface [brief | <interface-id> | vlan <vlan-id>]`
* `SW1# show ip interface brief | exclude unassigned`

# Forwarding Architecture

## Process switching

* software switching / slow path
  * `ip_input` process
    * packets coming to / going out from router itself
    * complicated packets, e.g., IP packets with Options
    * extra info needed, e.g., unresolved ARP entries
![](img/2024-09-18-11-24-30.png)

### Data structures

* RIB
  * to find next hop IP address and outgoing interface
* ARP table
  * to find next hop MAC address

## CEF

* Cisco Express Forwarding (CEF) / fast path
* [software CEF](#software-cef)
* [hardware CEF](#hardware-cef)
  * ASIC
  * TCAM (Tenary CAM)
  * NPU

### TCAM

* CAM result is binary (0 or 1)
* TCAM (0 true, 1 false, X don't care)

![](img/2024-09-18-17-07-56.png)

* The TCAM entries are stored in `Value`, `Mask`, and `Result` (VMR) format.
  * The `value` indicates the fields that should be searched, such as the IP address and protocol fields. 
  * The `mask` indicates the field that is of interest and that should be queried. 
  * The `result` indicates the action that should be taken with a match on the value and mask.
  * Multiple actions can be selected

### Distributed Forwarding

* If the line cards are equipped with forwarding engines so that they can make packet switching decisions without intervention of the RP, this is known as a *distributed forwarding architecture*.
* On the contrary, centralized forwarding will involve *route processor (RP)* to process every packet

### Software CEF

* also known as the software Forwarding Information Base (FIB)
* two tables
  * FIB
  * Adjacency table (next-hop IP address, next-hop MAC, egress if MAC)

![](img/2024-09-18-17-16-20.png)

* Software CEF in hardware-based platforms is not used to do packet switching as in software-based platforms; instead, it is used to program the hardware CEF.

### Hardware CEF

* NPU: Unlike ASICs, **NPUs are programmable**, and their firmware can be changed with relative ease.

### SDM templates

> The allocation ratios between the various TCAM tables are stored and can be modified with Switching Database Manager (SDM) templates.
* `[show] sdm prefer {vlan | advanced}`
  * then restart switch w/ `reload`
* Every switch in a switch stack must be configured with the same SDM template.

