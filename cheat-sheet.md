- [protocol numbers](#protocol-numbers)
- [addresses](#addresses)
  - [OSPF](#ospf)

# protocol numbers

| Protocol | Number |
|:--|:--|
| EIGRP | 88
| OSPF | 89
| IGMPv2 | 2
| PIM | 103
| IPSEC AH | 51
| IPSEC ESP | 50

# addresses

## OSPF

* AllSPFRouters: **224.0.0.5** or MAC address 01:00:5E:00:00:05
* AllDRouters: **224.0.0.6** or MAC address 01:00:5E:00:00:06
* v3
  * FF02::05: OSPFv3 AllSPFRouters
  * FF02::06: OSPFv3 AllDRouters