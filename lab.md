- [Lab: vlan](#lab-vlan)
- [Config l2 switch](#config-l2-switch)
  - [Task 1: Create VLAN's](#task-1-create-vlans)
    - [config VTP](#config-vtp)
    - [config vlan](#config-vlan)
  - [Task 2: Configure Trunks](#task-2-configure-trunks)
  - [Task 3: Assign VLANs](#task-3-assign-vlans)
  - [Task 4: Routing Between VLAN's](#task-4-routing-between-vlans)
    - [config vlan interface / SVI on switch](#config-vlan-interface--svi-on-switch)
    - [config a subinterface on router](#config-a-subinterface-on-router)
    - [config dhcp](#config-dhcp)
  - [Task 6: Internet Access (optional)](#task-6-internet-access-optional)
    - [on router](#on-router)
  - [misc.](#misc)
    - [config route port](#config-route-port)
- [Netflow](#netflow)
- [CoPP](#copp)
- [VRF](#vrf)
- [EtherChannel](#etherchannel)
- [MST](#mst)

# Lab: vlan

* https://user.pnetlab.com/store/labs/detail?id=16405723981793

![](img/2024-10-09-15-21-04.png)

# Config l2 switch

* Task 1: Create VLAN's
  * mainly VTP
* Task 2: Configure Trunks
  *  limit the vlan's (pruning) that cross the trunk
     *  `allowed vlan`
  *  by default Cisco permit that all the created vlan's go throught the trunk
     *  this could be a STP issue
* Task 3: Assign VLANs
  * mainly config access ports
* Task 4: Routing Between VLAN's
* Task 5: Static Routes (optional)
* Task 6: Internet Access (optional)

## Task 1: Create VLAN's

### config VTP

```
! on primary server
vtp domain mydomain
vtp version 3
vtp mode server
vtp password pnet
exit
vtp primary
> This system is becoming primary server for feature vlan 
> No conflicting VTP3 devices found.
> Do you want to continue? [confirm]
show vtp status

! on clients
vtp domain mydomain
vtp version 3
vtp mode client
vtp password pnet
show vtp status
```

### config vlan

```
vlan 200
name test
end
show vlan id 200
show spanning-tree vlan 200
! verify the vlan is allowed in trunk
show interface trunk
```

* all trunk port will join this vlan automatically

```
SW1#show vlan id 200

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
200  test                             active    Et0/0, Et0/1, Et0/2
```

## Task 2: Configure Trunks

```
switchport trunk encapulation dot1q
switchport mode trunk
switchport nonegotiate
switchport trunk allowed vlan 1,20,40,50,60,100
switchport trunk native vlan 999
```

## Task 3: Assign VLANs

```
interface e0/2
switchport mode access
switchport access vlan 200

! or 
! switchport host
```

* if vlan 200 not configured prior, it will be automatically created

## Task 4: Routing Between VLAN's

* so far, hosts in same vlan can talk to each other
* but for hosts in different vlans to talk to each other, we must goto layer 3 and start IP routing
  * router will be the default gw, and dhcp server

### config vlan interface / SVI on switch

```
interface VLAN70
 ip address 192.168.70.1 255.255.255.0
```

### config a subinterface on router

```
interface Ethernet0/0
 no shutdown
interface Ethernet0/0.10
 encapsulation dot1q 10
 ip address 192.168.10.1 255.255.255.0
 ip nat inside
interface Ethernet0/0.60
 encapsulation dot1q 60
 ip address 192.168.60.1 255.255.255.0
 ip nat inside
```

### config dhcp

```
ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp excluded-address 192.168.10.21 192.168.10.254
ip dhcp excluded-address 192.168.60.1 192.168.60.10
ip dhcp excluded-address 192.168.60.21 192.168.60.254

ip dhcp pool VLAN10
 network 192.168.10.0 /24
 ip default-router 192.168.10.1
 dns-server 8.8.8.8 

ip dhcp pool VLAN60
 network 192.168.60.0 /24
 ip default-router 192.168.60.1
 dns-server 8.8.8.8
```

## Task 6: Internet Access (optional)

### on router

```
! first configure inside/outside nat interfaces
!
interface ethernet 0/1
 ip nat outside
 ip address dhcp
 no shutdown

interface ethernet 0/0.10
 ip nat inside

inteface ethernet 0/0.60
 ip nat inside

! second, create an ACL to match traffic
!
access-list 1 permit 192.168.10.0 0.0.0.255
access-list 1 permit 192.168.60.0 0.0.0.255

! third, create a nat rule
!
ip nat inside source list 1 interface ethernet 0/1 overload
```

## misc.

### config route port

```
no switchport
ip address 192.168.100.1 255.255.255.0
```

# Netflow

```
flow record myRec
 match ipv4 protocol
 match ipv4 source address
 match ipv4 destination address
 match transport source-port
 match transport destination-port
 collect counter bytes
 collect counter packets
!
flow exporter myExp
 destination 10.1.1.99
 transport udp 2055
!
flow monitor netflow-mon
 exporter myExp
 record myRec
!
interface Ethernet0/1
 ip address 10.1.1.1 255.255.255.0
 ip flow monitor netflow-mon input
```

# CoPP

```
ip access-list extended ICMP-TRAFFIC
 permit icmp any any
!
class-map match-all CLASS-ICMP-TO-RP
 match access-group name ICMP-TRAFFIC
!
policy-map Policy-CoPP-ICMP
 class CLASS-ICMP-TO-RP
  police 12000 conform-action transmit  exceed-action drop  violate-action drop 
!
control-plane
 service-policy input Policy-CoPP-ICMP
```

# VRF

```
R1(config)# vrf definition MGMT
R1(config-vrf)# address-family ipv4
R1(config)# interface GigabitEthernet0/3
R1(config-if)# vrf forwarding MGMT
R1(config-if)# ip address 10.0.3.1 255.255.255.0
R1(config)# ip route vrf MGMT 0.0.0.0 0.0.0.0 10.10.1.1
```

# EtherChannel

```
DLS1(config)#int range e0/2, e2/2
DLS1(config-if-range)#switchport trunk encapsulation dot1q
DLS1(config-if-range)#switchport mode trunk
DLS1(config-if-range)#channel-group 1 mode on
Creating a port-channel interface Port-channel 1

DLS1(config-if-range)#exit
DLS1(config)#int range e0/0, e2/0
DLS1(config-if-range)#switchport trunk encapsulation dot1q
DLS1(config-if-range)#switchport mode trunk
DLS1(config-if-range)#channel-group 2 mode active
Creating a port-channel interface Port-channel 2

DLS1(config)#int range e0/1, e2/1
DLS1(config-if-range)#switchport trunk encapsulation dot1q
DLS1(config-if-range)#switchport mode trunk
DLS1(config-if-range)#channel-group 3 mode desirable
Creating a port-channel interface Port-channel 3

ALS2#show etherchannel summary
ALS1#show interfaces trunk
```

# MST

