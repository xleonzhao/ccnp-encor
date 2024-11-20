- [Fabric network](#fabric-network)
- [SD-Access](#sd-access)

# Fabric network

* an _overlay network_ (_virtual network_ [VN]) built over an _underlay network_ (physical network) using overlay tunneling technologies such as VXLAN.
* enabling 
  * host mobility
  * network automation
  * network virtualization
  * segmentation
* next-gen overlay fabric technologies
  * SD-Access
    * main components of the Cisco Digital Network Architecture (Cisco DNA)
      * intent-based networking in Cisco enterprise networks
    * designed only for enterprise campus and branch networks
  * SD-WAN
    * used to be MPLS, Dynamic Multipoint Virtual Private Network (DMVPN) or Intelligent WAN (IWAN)
    * recent changes
      * majority of enterprise traffic flows to public clouds and the Internet
    * SD-WAN fabric is a cloud-based WAN

# SD-Access

* problem to address
  * constant growth of users and endpoints
    * security policies
    * management overhead
  * manual config
  * mobility
  * trouble-shooting
* solutions
  * network automation
    * via DNA center
  * network assurance and analytics
    * telemetry
  * host mobility
  * identity services
    * Cisco Identity Services Engine (ISE)
  * policy enforcement
    * ACL not scalable
      * relying on IP addresses and subnets
    * Security Group Access Control Lists (SGACLs)
  * secure segmentation
    * guest
    * corporate
    * facilities
    * IoT
  * network virtualization
    * VRF / VN
    * each with a distinct set of access policies
* two main components
  * Cisco Campus fabric solution
    * managed via CLI/API using NETCONF/YANG
  * Cisco DNA Center
* arch.

![](img/2024-11-18-15-27-27.png)

