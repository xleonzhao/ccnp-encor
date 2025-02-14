- [Lab: vlan](#lab-vlan)
- [Config l2 switch](#config-l2-switch)
  - [Task 1: Create VLAN's](#task-1-create-vlans)
    - [config VTP](#config-vtp)
    - [config vlan](#config-vlan)
  - [Task 2: Configure Trunks](#task-2-configure-trunks)
  - [Task 3: Assign VLANs](#task-3-assign-vlans)
  - [Task 4: Routing Between VLAN's](#task-4-routing-between-vlans)
    - [config a subinterface on router](#config-a-subinterface-on-router)
    - [config DHCP](#config-dhcp)
  - [Task 6: Internet Access (optional)](#task-6-internet-access-optional)
    - [config NAT](#config-nat)
  - [misc.](#misc)
    - [config routed port](#config-routed-port)
- [Netflow](#netflow)
- [CoPP](#copp)
- [VRF](#vrf)
- [HSRP](#hsrp)
- [VRRP](#vrrp)
- [EtherChannel](#etherchannel)
- [MST](#mst)
- [BGP](#bgp)
- [OSPF](#ospf)
- [IP SLA and EEM](#ip-sla-and-eem)
- [NETCONF](#netconf)
  - [get it running](#get-it-running)
  - [use netconf](#use-netconf)
- [line](#line)
- [GRE](#gre)
- [IPSec](#ipsec)
  - [GRE/transport mode](#gretransport-mode)
  - [VTI/tunnel mode](#vtitunnel-mode)

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

### config a subinterface on router

```
interface Ethernet0/0
 no shutdown
interface Ethernet0/0.10
 encapsulation dot1q 10
 ip address 192.168.10.1 255.255.255.0
interface Ethernet0/0.60
 encapsulation dot1q 60
 ip address 192.168.60.1 255.255.255.0
```

### config DHCP

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

### config NAT

```
! first configure inside/outside nat interfaces
!
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

### config routed port

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

# HSRP

```
SW2(config)# interface vlan 10
SW2(config-if)# ip address 172.16.10.2 255.255.255.0
SW2(config-if)# standby 10 ip 172.16.10.1
SW2(config-if)# standby 10 preempt
```

# VRRP

```
R2(config)# interface GigabitEthernet 0/0
R2(config-if)# ip address 172.16.20.2 255.255.255.0
R2(config-if)# vrrp 20 ip 172.16.20.1
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

```
! make self be the root for mst 1
DLS1(config)#spanning-tree mst 1 root primary
DLS2(config)#spanning-tree mst configuration
DLS2(config-mst)#name CCNP
DLS2(config-mst)#revision 0
! distribute spanning-tree to VLANs
DLS2(config-mst)#instance 1 vlan 100, 200
DLS2(config-mst)#instance 2 vlan 300, 400
DLS2(config-mst)#instance 3 vlan 500, 600
DLS2(config-mst)#instance 4 vlan 700, 800
DLS1(config)#spanning-tree mode mst
! check
DLS1#show spanning-tree mst 1
```

# BGP

```
router bgp <ASN>
  neighbor <ip> remote-as <ASN>
  neighbor <ip> update-source Loopback0 
  address-family ipv4
    network <ip> mask <mask>
```

# OSPF

```
router ospf 1
  router-id 1.1.1.1
  network <interface ip address> <interface ip mask>
```

# IP SLA and EEM

```
! SLA
R2(config)#ip sla 1
R2(config-ip-sla)#icmp-echo 10.1.23.3 source-ip 10.1.23.2
R2(config-ip-sla-echo)#frequency 10
R2(config-ip-sla-echo)#exit
R2(config)#ip sla schedule 1 life forever start-time now
!
! tracking
track 1 ip sla 1 reachability
!
! EEM
!   when link A down, use link B
R2(config)#event manager applet TRACK_DOWN
R2(config-applet)#event track 1 state down
R2(config-applet)#action 1.0 cli command "enable"
R2(config-applet)#action 2.0 cli command "conf t"
R2(config-applet)#action 3.0 cli command "ip route 8.8.8.8 255.255.255.255 10.1.24.4"
R2(config-applet)#action 4.0 cli command "no ip route 8.8.8.8 255.255.255.255 10.1.23.3"
!
! optional: debug EEM
R2#debug event manager action cli
```

* another EEM example

```
R1#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#event manager applet save-and-log
R1(config-applet)#event syslog pattern "%SYS-5-CONFIG_I: Configured from "
R1(config-applet)#action 10 cli command "enable"
R1(config-applet)#action 20 cli command "show clock | append unix:config-changes"
R1(config-applet)#action 30 cli command "show start | append unix:config-changes"
R1(config-applet)#action 40 cli command "write memory"
R1(config-applet)#action 50 cli command "show start | append unix:config-changes"
R1(config-applet)#end
```

# NETCONF

* https://user.pnetlab.com/store/labs/detail?id=16000809382486
* https://community.cisco.com/t5/networking-blogs/getting-started-with-netconf-yang-part-1/ba-p/3661241

## get it running

* csr1000v image not found
  * use [this one](https://dl.nextadmin.net/dl/EVE-NG-image/qemu/csr1000vng-universalk9.16.09.01.Fuji.tar.gz)
* netconf keysign error: `NETCONF/SSH: error: ssh_r sa_sign: RSA_sign failed`
  * fix:
  ```
  conf t
  no netconf-yang
  crypto key generate rsa modulus 2048
  netconf-yang
  ```
* connect Ubuntu-Server via an external terminal
  * find ubuntu-server interface ip address: `ip a`
  * login to Pnetlab VM
  * then ssh to ubuntu-server
  * copy and paste works better this way
* check session: `show netconf-yang sessions`

## use netconf

* send HELLO 
  * netconf will not respond to hello
```
<hello xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
<capabilities>
 <capability>urn:ietf:params:netconf:base:1.0</capability>
</capabilities>
</hello>
]]>]]>
```
* send RPC
```
<rpc message-id="103" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
<get>
 <filter>
 <interfaces xmlns="urn:ietf:params:xml:ns:yang:ietf-interfaces"/>
 </filter>
</get>
</rpc>
]]>]]>
```

# line

```
username cisco secret cisco
hostname R1
ip domain-name cisco.com
crypto key generate rsa
access-list 10 permit 192.168.100.1 255.255.255.0
!
line con
  exec-timeout 5 0
  login local
line vty 0 9
  exec-timeout 5 0
  transport input ssh
  access-class 10 in
  login local
```

# GRE

```
R(config)#interface tunnel 0
BR(config-if)#ip address 172.16.34.4 255.255.255.240
BR(config-if)#tunnel source ethernet 0/0
BR(config-if)#tunnel destination 172.16.23.3
```

# IPSec

## GRE/transport mode

```
crypto isakmp policy 10
 encr aes 256
 hash sha512
 authentication pre-share
 group 15
 lifetime 7200
crypto isakmp key cisco123 address 172.16.23.3    
!
crypto ipsec transform-set AES_SHA esp-aes 256 esp-sha512-hmac 
 mode transport
!
crypto ipsec profile IPSEC_PROFILE
 set transform-set AES_SHA 

interface Tunnel0
 ip address 172.16.34.4 255.255.255.240
 tunnel source Ethernet0/0
 tunnel destination 172.16.23.3
 tunnel protection ipsec profile IPSEC_PROFILE
```

## VTI/tunnel mode

```
crypto isakmp policy 10
 encr aes 256
 hash sha512
 authentication pre-share
 group 15
 lifetime 7200
crypto isakmp key cisco123 address 172.16.23.3    
!
crypto ipsec transform-set AES_SHA esp-aes 256 esp-sha512-hmac 
 mode tunnel <-- diff
!
crypto ipsec profile IPSEC_PROFILE
 set transform-set AES_SHA 

interface Tunnel0
 ip address 172.16.34.4 255.255.255.240
 tunnel source Ethernet0/0
 tunnel mode ipsec ipv4 <-- add
 tunnel destination 172.16.23.3
 tunnel protection ipsec profile IPSEC_PROFILE
```