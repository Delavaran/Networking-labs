
# OSPF Scalability and Stub Areas

## OSPF Scalability Factors

Scalability in OSPF is mainly affected by two factors:

1. **Topology complexity**

   * How many routers are in the area.
   * Larger flooding domains mean more frequent SPF runs.

2. **Reachability information**

   * How many routes are being advertised.
   * Larger routing tables take longer to flood and process.

Scalability in OSPF is achieved by minimizing both of these factors.

---

## OSPF Areas and Summarization

* **Topology summarization** is achieved through OSPF areas.

  * Hides the details of the topology within an area.
  * SPF runs only for intra-area adjacencies.
  * However, areas do not hide reachability information.

* **NLRI (Network Layer Reachability Information) summarization** reduces the number of routes.

  * Takes multiple longer-prefix matches and combines them into a smaller match.
  * Due to OSPF being a link-state protocol, summarization can only be done on ABRs.

Two approaches to summarization:

1. **Per-prefix basis**
2. **Per-LSA summarization**

   * Achieved using stub areas (removing inter-area routes and replacing them with a default route).
   * The level of summarization depends on the type of stub area used.

---

## How OSPF Stub Areas Work

* Filtering is enforced at common transit points of the OSPF topology — the ABRs.
* The ABR controls which LSAs enter the area:

  * Types 3, 4, and 5 are filtered depending on the stub type.
* Reachability information is removed and replaced with a default route.
* Reachability still exists via the default route.

> **Important:** All routers in the area must agree on the stub flag because it is part of adjacency negotiation.

---

## OSPF Stub Area Types

1. **Stub Area**

   * Filters **external routes** (LSA 4 and 5).
   * ABR injects a default route (Type-3).

   **Logic:**

   * I know how to get to my ABR.
   * My ABR knows how to get to the ASBR.
   * The ASBR knows how to reach the external route.
   * → If I default to the ABR, I don’t need the specific external routes.

   **Result:**

   * ABR removes LSA 4 and 5.
   * ABR originates a default route (LSA Type-3).

---

2. **Totally Stubby Area**

   * Filters **external and inter-area routes** (LSA 3, 4, 5).
   * ABR injects only a default route (Type-3).

   **Logic:**

   * Same as stub, but applies to both external and inter-area routes.
   * Since the ABR knows all inter-area and external destinations, the routers inside can just use a default route.

   **Result:**

   * ABR removes LSA 3, 4, 5.
   * ABR originates a default route (LSA Type-3).

   **Use Cases:**

   * Commonly used in DMVPN topologies.
   * Best suited for designs with a single exit point.

---

3. **NSSA (Not-So-Stubby Area)**

   * Stub areas block external routes — but what if redistribution into the stub is needed?
   * NSSA allows redistribution while still blocking other external routes.

   **Result:**

   * Redistributing router generates **LSA 7 (NSSA External)**.
   * ABR translates LSA 7 into LSA 5 for Area 0.
   * ABR removes LSA 4 and 5.
   * ABR does **not** automatically generate a default route.

   **Metric Types:**

   * LSA 7 uses **N1** and **N2** (similar to E1/E2 in Type-5).

   **Note:**

   * A default route can be configured manually.
   * This provides flexibility for traffic engineering (e.g., hot-potato vs. cold-potato routing).

---

4. **Totally NSSA (Not-So-Totally-Stubby Area)**

   * Combination of Totally Stubby and NSSA.
   * Filters LSA 3, 4, 5, but allows redistribution.

   **Result:**

   * Redistributing router generates LSA 7 (NSSA External).
   * ABR translates LSA 7 into LSA 5 for Area 0.
   * ABR removes LSA 3, 4, 5.
   * ABR injects a default route.

---

## Configuration Examples

### Stub Area
![OSPF Topology](topology.png)

on Router 5,8,10

```bash
router ospf 1
 area 1 stub
```

### Totally Stubby Area

```bash
router ospf 1
 area 2 stub no-summary
```
the no-summary keyword is not exchanged in the adjacancy process, so there is no need for it to be configured on none Abr routers in a stub area .

---

## Verification Example (R8)

### before stub :


```bash

R8#sh ip route ospf 
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

      172.16.0.0/16 is variably subnetted, 23 subnets, 2 masks
O IA     172.16.1.0/24 [110/50] via 172.16.16.5, 00:02:51, Ethernet0/1
O IA     172.16.2.0/24 [110/50] via 172.16.16.5, 00:02:51, Ethernet0/1
O IA     172.16.3.0/24 [110/40] via 172.16.16.5, 00:03:14, Ethernet0/1
O IA     172.16.4.0/24 [110/30] via 172.16.16.5, 00:03:14, Ethernet0/1
O IA     172.16.5.0/24 [110/40] via 172.16.16.5, 00:03:14, Ethernet0/1
O IA     172.16.6.0/24 [110/30] via 172.16.16.5, 00:03:14, Ethernet0/1
O IA     172.16.7.0/24 [110/20] via 172.16.16.5, 00:03:14, Ethernet0/1
O IA     172.16.8.0/24 [110/30] via 172.16.16.5, 00:03:14, Ethernet0/1
O IA     172.16.9.0/24 [110/20] via 172.16.16.5, 00:03:14, Ethernet0/1
O IA     172.16.10.0/24 [110/20] via 172.16.16.5, 00:03:14, Ethernet0/1
O IA     172.16.11.0/24 [110/30] via 172.16.16.5, 00:01:15, Ethernet0/1
O IA     172.16.12.0/24 [110/30] via 172.16.16.5, 00:03:14, Ethernet0/1
O        172.16.13.0/24 [110/20] via 172.16.15.10, 00:02:24, Ethernet0/0
O        172.16.14.0/24 [110/20] via 172.16.15.10, 00:02:24, Ethernet0/0
O IA     172.16.19.0/24 [110/40] via 172.16.16.5, 00:03:14, Ethernet0/1
      198.22.10.0/32 is subnetted, 7 subnets
O IA     198.22.10.1 [110/31] via 172.16.16.5, 00:03:14, Ethernet0/1
O IA     198.22.10.2 [110/21] via 172.16.16.5, 00:03:14, Ethernet0/1
O IA     198.22.10.4 [110/21] via 172.16.16.5, 00:03:14, Ethernet0/1
O IA     198.22.10.7 [110/31] via 172.16.16.5, 00:03:14, Ethernet0/1
O IA     198.22.10.9 [110/41] via 172.16.16.5, 00:02:51, Ethernet0/1
O        198.22.10.10 [110/11] via 172.16.15.10, 00:02:24, Ethernet0/0
R8#

```

## after changing area 1 into stub

```bash
R8#sh ip route ospf 
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override, p - overrides from PfR

Gateway of last resort is 172.16.16.5 to network 0.0.0.0

O*IA  0.0.0.0/0 [110/11] via 172.16.16.5, 00:00:01, Ethernet0/1
      172.16.0.0/16 is variably subnetted, 10 subnets, 2 masks
O        172.16.13.0/24 [110/20] via 172.16.15.10, 00:00:38, Ethernet0/0
O        172.16.14.0/24 [110/20] via 172.16.15.10, 00:00:38, Ethernet0/0
      198.22.10.0/32 is subnetted, 2 subnets
O        198.22.10.10 [110/11] via 172.16.15.10, 00:00:38, Ethernet0/0
R8#
```

```bash
R8# show ip ospf database
OSPF Router with ID (8.8.8.8) (Process ID 1)

Router Link States (Area 1)
  Link ID         ADV Router      Age     Seq#       Checksum Link count
  5.5.5.5         5.5.5.5         670     0x8000000C 0x0079D7 1
  8.8.8.8         8.8.8.8         1454    0x8000000D 0x00059F 5
  10.10.10.10     10.10.10.10     1700    0x8000000B 0x00A9B3 4

Summary Net Link States (Area 1)
  Link ID         ADV Router      Age     Seq#       Checksum
  0.0.0.0         5.5.5.5         1196    0x8000000A 0x000918
```

---

## ospf configuration of each router :
### R5 :
```bash
R5#sh run | sec ospf 
 ip ospf 1 area 1
 ip ospf 1 area 0
 ip ospf 1 area 0
 ip ospf 1 area 0
router ospf 1
 router-id 5.5.5.5
 area 1 stub no-summary
R5#
```

### R8 :

```bash
R8#sh run | sec ospf 
router ospf 1
 router-id 8.8.8.8
 area 1 stub
 network 0.0.0.0 255.255.255.255 area 1
R8#
```

### R10 :

```bash
R10#sh run | sec ospf 
router ospf 1
 router-id 10.10.10.10
 area 1 stub
 network 0.0.0.0 255.255.255.255 area 1
R10#
```






