# OSPF - Other Summary Application  

![Topology](ospf-topology.png)

## 1) Traffic Engineering
* OSPF (like IP routing in general) prefers the most specific prefix (longest match) over a less specific one, regardless of cost
* Lets say in area 2 we want to decide how R7 would reach R5's loopback of 50.50.50.50 

currently R7 would go through R3 because of the lower metric :
```
R7#sh ip ospf database summary 50.50.50.50

            OSPF Router with ID (7.7.7.7) (Process ID 1)

                Summary Net Link States (Area 2)

  LS age: 121
  Options: (No TOS-capability, DC, Upward)
  LS Type: Summary Links(Network)
  Link State ID: 50.50.50.50 (summary Network Number)
  Advertising Router: 3.3.3.3
  LS Seq Number: 80000001
  Checksum: 0x98C4
  Length: 28
  Network Mask: /32
        MTID: 0         Metric: 11 

  LS age: 121
  Options: (No TOS-capability, DC, Upward)
  LS Type: Summary Links(Network)
  Link State ID: 50.50.50.50 (summary Network Number)
  Advertising Router: 6.6.6.6
  LS Seq Number: 80000001
  Checksum: 0xA2A4
  Length: 28
  Network Mask: /32
        MTID: 0         Metric: 21 

R7#
```

but lets say we want our traffic to go through R6, then we should stop R3 from advertising  this prefix but the downside is that we would lose redundancy.
so we can make R3 to advertise a less specific prefix, R6 loopback is 50.``50.50.50/32``, we can advertise ``50.50.50.50/31`` on R3 which would include ``50.50.50.50`` and ``50.50.50.51`` .
**on R3**
```
area 0 range 50.50.50.0 255.255.255.254
```
**now on R7**
```
R7#sh ip ospf database summary 50.50.50.50

            OSPF Router with ID (7.7.7.7) (Process ID 1)

                Summary Net Link States (Area 2)

  LS age: 440
  Options: (No TOS-capability, DC, Upward)
  LS Type: Summary Links(Network)
  Link State ID: 50.50.50.50 (summary Network Number)
  Advertising Router: 3.3.3.3
  LS Seq Number: 80000002
  Checksum: 0x90CC
  Length: 28
  Network Mask: /31
        MTID: 0         Metric: 11 

  LS age: 983
  Options: (No TOS-capability, DC, Upward)
  LS Type: Summary Links(Network)
  Link State ID: 50.50.50.50 (summary Network Number)
  Advertising Router: 6.6.6.6
  LS Seq Number: 80000001
  Checksum: 0xA2A4
  Length: 28
  Network Mask: /32
        MTID: 0         Metric: 21 

R7# 
```

```
R7#sh ip route ospf 
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override, p - overrides from PfR

Gateway of last resort is not set

      50.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
O IA     50.50.50.50/31 [110/21] via 172.16.4.3, 00:09:41, Ethernet0/2
O IA     50.50.50.50/32 [110/31] via 172.16.5.6, 00:09:41, Ethernet0/1
      172.16.0.0/16 is variably subnetted, 17 subnets, 2 masks
O        172.16.1.0/24 [110/20] via 172.16.3.9, 00:22:36, Ethernet0/0
O        172.16.2.0/24 [110/20] via 172.16.3.9, 00:22:36, Ethernet0/0
O IA     172.16.6.0/24 [110/20] via 172.16.4.3, 00:21:37, Ethernet0/2
```
* While technically valid, summarizing a /32 into a /31 to influence path selection is uncommon in production. Typically, route-maps or cost manipulation are used.


---


## 2) Filter routes
it's done using this keywords with the summary commands :
``area x range not-advertised`` and
``summary-address not-advertised``

lets say in the same topology, we want to prevent the advertisemnt of R2's loopback towards Area 1.

**Currently on R8**
```
R8#sh ip route 198.22.10.2
Routing entry for 198.22.10.2/32
  Known via "ospf 1", distance 110, metric 21, type inter area
  Last update from 172.16.16.5 on Ethernet0/1, 00:03:36 ago
  Routing Descriptor Blocks:
  * 172.16.16.5, from 5.5.5.5, 00:03:36 ago, via Ethernet0/1
      Route metric is 21, traffic share count is 1
R8#
```
**on R5**
```
R5(config-router)#area 0 range 198.22.10.2 255.255.255.255 not-advertise 
```
**R8 rib after the change**
```
R8#sh ip route 198.22.10.2
% Subnet not in table
R8#
```

---


## 3) Changing the Cost
* For **inter-Area** prefixes we can change the cost, witht the ``cost`` key-word in the ``area x range`` command 
* Lets say in area 2 we want to decide how R7 would reach R5's loopback of 50.50.50.50 

**R7's Rib before the change**
```
R7#sh ip ospf database summary 50.50.50.50

            OSPF Router with ID (7.7.7.7) (Process ID 1)

                Summary Net Link States (Area 2)

  LS age: 30
  Options: (No TOS-capability, DC, Upward)
  LS Type: Summary Links(Network)
  Link State ID: 50.50.50.50 (summary Network Number)
  Advertising Router: 3.3.3.3
  LS Seq Number: 80000004
  Checksum: 0x92C7
  Length: 28
  Network Mask: /32
        MTID: 0         Metric: 11 

  LS age: 481
  Options: (No TOS-capability, DC, Upward)
  LS Type: Summary Links(Network)
  Link State ID: 50.50.50.50 (summary Network Number)
  Advertising Router: 6.6.6.6
  LS Seq Number: 80000002
  Checksum: 0xA0A5
  Length: 28
  Network Mask: /32
        MTID: 0         Metric: 21 

R7#
```
**On R3**
```
R3(config-router)#area 0 range 50.50.50.50 255.255.255.255 cost 30000
```

**R7's Rib after change**
```
R7#sh ip ospf database summary 50.50.50.50

            OSPF Router with ID (7.7.7.7) (Process ID 1)

                Summary Net Link States (Area 2)

  LS age: 12
  Options: (No TOS-capability, DC, Upward)
  LS Type: Summary Links(Network)
  Link State ID: 50.50.50.50 (summary Network Number)
  Advertising Router: 3.3.3.3
  LS Seq Number: 80000006
  Checksum: 0x239A
  Length: 28
  Network Mask: /32
        MTID: 0         Metric: 30000 

  LS age: 596
  Options: (No TOS-capability, DC, Upward)
  LS Type: Summary Links(Network)
  Link State ID: 50.50.50.50 (summary Network Number)
  Advertising Router: 6.6.6.6
  LS Seq Number: 80000002
  Checksum: 0xA0A5
  Length: 28
  Network Mask: /32
        MTID: 0         Metric: 21 

R7#
```
* This is useful when you want to influence inter-area route selection without touching intra-area metrics (which always win).

---

## 4) Adding tags
* unlike eigrp that we could attach tags for every prefix, in ospf we only can set tags for external routes, either at the point of redistribution or with the ``summary-address`` command .
* then we could match on these tags on a route-map and apply our policy to those specific prefixes 

```
summary-address 10.0.0.0 255.0.0.0 tag 123456
```
* Tags are not used in OSPF path selection directly, but allow policy decisions when redistributing into other protocols (e.g., BGP, EIGRP, another OSPF process).

---

## 5) Enforce area-local scope for NSSA routes
* we could set the p bit in type-7 lsa's using ``summary-address`` command with ``nssa-only`` keyword, as we dicussed in nssa section 
```
summary-address 150.22.33.9 255.255.255.255 nssa-only
```
* now the 150.22.33.9/32 prefix would not be advertised beyond the Nssa.

* With nssa-only, the Type-7 LSA is generated without the P-bit set, preventing translation into Type-5 by ABRs — keeping the route inside the NSSA.


