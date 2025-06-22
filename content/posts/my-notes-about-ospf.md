---
title: 'My notes about OSPF'
date: 2023-02-22T14:07:09-03:00
draft: false
ShowToC: true
TocOpen: true
cover:
    image: /my-notes-about-ospf/ospf_cover.png
    alt: 'My notes about OSPF'
    caption: 'My notes about OSPF'
---

# Introduction

This post is a summary of OSPF that I compiled during my CCIE journey, gathering information from RFCs, books, Cisco documentation, blogs, and other sources.

> **_IMPORTANT:_** I’m still in the process of converting this page from my original notes, so some information may be missing, and the formatting may not yet be fully refined.

------

# OSPFv2

OSPFv2 is documented under **RFC 2328**, which states the following:

OSPF routes IP packets based solely on the destination IP address found in the IP packet header.  IP packets are routed "as is" -- they are not encapsulated in any further protocol headers as they transit the Autonomous System.  OSPF is a dynamic routing protocol. It quickly detects topological changes in the AS (such as router interface failures) and calculates new loop-free routes after a period of convergence. 

This period of convergence is short and involves a minimum of routing traffic.

In a link-state routing protocol, each router maintains a database describing the Autonomous System's topology.  This database is referred to as the link-state database. Each participating router has an identical database.  Each individual piece of this database is a particular router's local state (e.g., the router's usable interfaces and reachable neighbors). 

The router distributes its local state throughout the Autonomous System by flooding.


## OSPFv2 Packet Header

![OSPFv2 Packet Header](/my-notes-about-ospf/packet_header.png)


# OSPFv3

OSPFv3 is documented under **RFC 5340**

Abstract:
> This document describes the modifications to OSPF to support version
   6 of the Internet Protocol (IPv6).  The fundamental mechanisms of
   OSPF (flooding, Designated Router (DR) election, area support, Short
   Path First (SPF) calculations, etc.) remain unchanged.  However, some
   changes have been necessary, either due to changes in protocol
   semantics between IPv4 and IPv6, or simply to handle the increased
   address size of IPv6.  These modifications will necessitate
   incrementing the protocol version from version 2 to version 3.  OSPF
   for IPv6 is also referred to as OSPF version 3 (OSPFv3).
   
> Changes between OSPF for IPv4, OSPF Version 2, and OSPF for IPv6 as
   described herein include the following.  Addressing semantics have
   been removed from OSPF packets and the basic Link State
   Advertisements (LSAs).  New LSAs have been created to carry IPv6
   addresses and prefixes.  OSPF now runs on a per-link basis rather
   than on a per-IP-subnet basis.  Flooding scope for LSAs has been
   generalized.  Authentication has been removed from the OSPF protocol
   and instead relies on IPv6's Authentication Header and Encapsulating
   Security Payload (ESP).

> Even with larger IPv6 addresses, most packets in OSPF for IPv6 are
   almost as compact as those in OSPF for IPv4.  Most fields and packet-
   size limitations present in OSPF for IPv4 have been relaxed.  In
   addition, option handling has been made more flexible.

> All of OSPF for IPv4's optional capabilities, including demand
   circuit support and Not-So-Stubby Areas (NSSAs), are also supported
   in OSPF for IPv6.


## OSPFv3 Packet Header

![OSPFv3 Packet Header](/my-notes-about-ospf/packet_header_ospfv3.png)

------

# Multicast Groups

## OSPFv2 Multicast groups
| Multicast Group | Description      |
|-----------------|------------------|
| 224.0.0.5       | All OSPF Routers |
| 224.0.0.6       | All DR Routers   |

## OSPFv3 Multicast groups
| Multicast Group | Description                 |
|-----------------|-----------------------------|
| FF02::5         | OSPFv3 Routers              |
| FF02::6         | OSPFv3 Designated Routers   |


------


# Timers
| Interval   | Broadcast | Non-Broadcast  | Point-to-Point  | Point-to-Multipoint  | Point-to-Multipoint Non-Broadcast |
|------------|-----------|----------------|-----------------|----------------------|-----------------------------------|
| Hello      | 10        | 30             | 10              | 30                   | 30                                |
| Dead       | 40        | 120            | 40              | 90                   | 120                               |
| Wait       |           |                | 40              | 90                   |                                   |
| Retransmit | 5         | 5              | 5               | 5                    | 5                                 |


------


# OSPFv2 LSA Types

## LSA Type 1 – Router LSA
- **Route Type:** Intra-Area  
- **CLI Command:**  
  `show ip ospf database router`  
- **Description:**  
  - Each router within an area floods a Type 1 Router LSA to other routers in the same area.  
  - This LSA describes all of the router’s interfaces, neighbor relationships, and metrics.

## LSA Type 2 – Network LSA
- **Route Type:** Intra-Area  
- **CLI Command:**  
  `show ip ospf database network`  
- **Description:**  
  - Generated by the **Designated Router (DR)** on multi-access networks (e.g., Ethernet).  
  - Represents the multi-access network and all attached routers as a **pseudonode** (virtual router).  
  - Lists all routers connected to the network segment, including the DR itself.  
  - Like Type 1 LSAs, Type 2 LSAs are flooded only within the originating area.

## LSA Type 3 – Network Summary LSA
- **Route Type:** Inter-Area  
- **CLI Command:**  
  `show ip ospf database summary`  
- **Description:**  
  - Originated by **ABRs**.  
  - Advertises destinations **outside** the local area **into** the area.  
  - Used by ABRs to inform internal routers about reachable destinations in other areas.  
  - ABRs also advertise routes from their own areas **into the backbone (Area 0)** using Type 3 LSAs.  
  - Can also be used to advertise **default routes** internal to the OSPF autonomous system.

## LSA Type 4 – ASBR Summary LSA
- **Route Type:** Inter-Area  
- **CLI Commands:**  
  `show ip ospf database asbr-summary`  
  `show ip ospf border-routers`  
- **Description:**  
  - Also originated by **ABRs**.  
  - Similar to Type 3 LSAs but specifically advertises routes to **ASBRs**.  
  - The destination is always a **host address** representing the ASBR router itself.

## LSA Type 5 – External LSA
- **Route Type:** External Routes (E1/E2)  
- **CLI Commands:**  
  `show ip ospf database external`  
  `show ip ospf border-routers`  
- **Description:**  
  - Originated by **ASBRs**.  
  - Advertises destinations **external** to the OSPF autonomous system (e.g., routes from BGP, static).  
  - Can also advertise **default routes** from outside the OSPF domain.  
  - Flooded throughout the **entire OSPF domain**.

## LSA Type 7 – NSSA External LSA
- **Route Type:** External Routes (N1/N2)  
- **CLI Commands:**  
  `show ip ospf database external`  
  `show ip ospf border-routers`  
- **Description:**  
  - Originated by **ASBRs within Not-So-Stubby Areas (NSSA)**.  
  - Similar to Type 5 LSAs but scoped only within the **originating NSSA**.  
  - Not flooded beyond the NSSA; may be **translated to Type 5** by the ABR for propagation beyond the NSSA.


------

# OSPF Options

## Down Bit
- The down bit helps prevent routing loops between MP-BGP and OSPF when LSA Type 3 are used, but not when External Routes are announced.
- With LSA Type 5 and Type 7, there is a new field checked to avoid loops is called the Tag field. When a PE redistributes a route from MP-BGP into OSPF as LSA Type 5 or LSA Type 7, it adds a Tag to the route (tag 3989725929 by default). So if another PE receives an LSA Type 5 or LSA Type 7 with this tag, it doesn’t redistribute it back into MP-BGP.  The default Tag value of 3989725929 can be changed to any other value (redistribute bgp 65001 subnets tag 100).

## N Bit
- This bit is used in hello packets for OSPF NSSA routers. When the N-bit is not supported, the routers won’t become neighbors.

## P Bit
- The P-bit (P stands for propagate) is set in the Options field of an LSA type 7. 
- P-bit tells the ABR if the LSA type 7 should be translated into a LSA type 5 or not.
- P-bit is automatically set by default for all prefixes that are redistributed by the ASBR and are flooded within that area only. 
- Can be disabled with nssa-only keyword.
- The P-bit is not set only when the NSSA ASBR and NSSA ABR are the same router for the area. 
- This P-bit is automatically set by the NSSA ABR (also the Forwarding Address (FA) is copied from Type 7 LSA). 

**Note:**
- If a router is attached to another AS and is also an NSSA ABR, it may originate a both a Type 5 and a Type 7 LSA for the same network. 
- The Type 5 LSA will be flooded to the backbone and the Type 7 will be flooded into the NSSA. If this is the case, the P-bit must be reset (P=0) in the Type 7 LSA so the Type 7 LSA isn’t again translated into a Type 5 LSA by another NSSA ABR.

## Routing Bit
- The routing bit is not a part of the LSA itself. Routing Bit is an internal maintenance bit used by IOS indicating that the route to the destination advertised by this LSA is valid. So when you see "Routing Bit Set on this LSA," it means that the route to this destination is in the routing table.


------


## Demand Circuit
- Per RFC 1793, Extending OSPF to Support Demand Circuits,  “OSPF Hellos and the refresh of OSPF routing information are suppressed on demand circuits, allowing the underlying data-link connections to be closed when not carrying application traffic.”  This feature allows low-speed and pay-per-use links, such as analog dial and ISDN, to run OSPF without the need for periodic hellos and LSA flooding. Periodic hellos are only suppressed for point-to-point and point-to-multipoint OSPF network types. 
- This feature is enabled with the interface-level command ip ospf demand-circuit and is negotiated as part of the neighbor adjacency establishment, thus only one OSPF router on the segment requires that the feature be enabled. If routers on the segment do not support it, it will just ignore the option in the HELLO packet, but OSPF neighbors will still be established. 

## Flood Reduction
- Per RFC 2328, OSPF Version 2, “an LSA's LS age is never incremented past the value MaxAge.", so when the Link State Age reaches MaxAge, "the router must attempt to flush the LSA... by reflooding the MaxAge LSA just as if it was a newly originated LSA". 
- In Cisco’s IOS implementation of OSPF, the MaxAge value is 3600 seconds, or 60 minutes. To ensure that an LSA is not aged out, which means it will be flushed from the OSPF database, each LSA is reflooded after 30 minutes, regardless of whether the topology is stable or not. This periodic flooding behavior is commonly referred to as the “paranoid update.” 
- The ip ospf flood-reduction feature stops unnecessary LSA flooding by setting the DoNotAge (DNA) bit in the LSA, removing the requirement for the periodic refresh. This needs to be enabled on links with OSPF neighbors attached, as on the other links, as there are no neighbors, no LSAs are sent anyways.

## Stub Router Feature
- Prevents a router from becoming a transit router (nontransit router). 
- Nontransit routers only forward packets to and from locally attached subnets.

max-metric router-lsa 
- only transit networks
max-metric router-lsa include-stub
- transit networks + stub networks (ex: loopback)
max-metric router-lsa on-startup announce-time 
max-metric router-lsa on-startup wait-for-bgp

## OSPF Graceful Shutdown
Similar to using the shutdown command on an  interface, there is also a shutdown command for OSPF. This allows you to  gracefully stop the OSPF routing process without removing the  configuration from your router. You can do this globally or on the  interface level.

If you want to do a graceful shutdown globally, you have to use the shutdown command under the OSPF process. This will:
- Drop all neighbor adjacencies
- Flush all LSAs that the router originated by setting the age to 3600 seconds
- Send hello packets with the DR/BDR set to 0.0.0.0 and an empty neighbor list. This will trigger other OSPF routers to fall back to init state.
- Stop sending/receiving OSPF packets

If you shut OSPF on the interface level with the ip ospf shutdown command, it will do this:
- Drop all neighbor adjacencies on the interface you selected
- Flood updated LSAs where all information (prefix, neighbors) from the selected interface is removed
- Send hello packets with the DR/BDR set to 0.0.0.0 and an empty neighbor list. This will trigger other OSPF router to fall back to init state
- Stop sending/receiving OSPF packets


------


# OSPF Filtering

## OSPF RIB Filtering
- distribute-list with ACL
- distribute-list with route-map
- Administrative Distance
- RIB filtering does not stop the flooding of LSAs within the area

## Inter Area Routes
- area X range 10.0.0.0 not-advertise
	- Only effective when ABR is translating from Type 1 LSA to a Type 3 LSA (learn from INTRA Area and advertise INTER Area)
- area X filter-list prefix XXX in
	- prevent routes from being advertised TO the specified area
- area X filter-list prefix XXX out
	- prevent routes from being advertised FROM the specified area

## External Routes (LSA Type 5/7)
Effective to filter when going between areas
Has to be applied on the device that is doing the translation into Type 5 LSA
- summary-address 10.0.0.0 255.255.255.0 not-advertise
- summary-address 10.0.0.0 255.255.255.0 nssa-only


------


# Inter-Area OSPF is Distance Vector (OSPF Loop Prevention)

The content below is from an amazing post byy **Jeff Doyle**, available on the link below, where Jeff gets into details about OSPF's behavior with Intra-Area deployments.

> **_Source:_** https://www.networkworld.com/article/2348778/my-favorite-interview-question.html

**Why does OSPF require all traffic between non-backbone areas to pass through a backbone area (area 0)?**

Comparing  three fundamental concepts of link state protocols, concepts that even most OSPF beginners understand, easily derives the answer to the above question. 

**The first concept:**

Every link state router floods information about itself, its links, and its neighbors to every other router. From this flooded information each  router builds an identical link state database. Each router then independently runs a shortest-path-first calculation on its database – a local calculation using distributed information – to derive a shortest-path tree. This tree is a sort of map of the shortest path to every other router.

One of the advantages of link state protocols is that the link state database provides a “view” of the entire network, preventing most routing loops. This is in contrast to distance vector protocols, in which route information is passed hop-by-hop through the network and a calculation is performed at each hop – a distributed calculation using local information. Each router along a route is dependent on the router before it to perform its calculations correctly and then correctly pass along the results. When a router advertises the prefixes it learns to its neighbors it’s basically saying, “I know how to reach these destinations.” And because each distance vector router knows only what its neighbors tell it, and has no “view” of the network beyond the neighbors, the protocol is vulnerable to loops.

**The second concept:**

When link state domains grow large, the flooding and the resulting size of the link state database becomes a scaling problem. The problem is remedied by breaking the routing domain into areas: That first concept is modified so that flooding occurs only within the boundaries of an area, and the resulting link state database contains only information from the routers in the area. This, in turn, means that each router’s calculated shortest-path tree only describes the path to other routers within the area.

**The third concept:**

OSPF areas are connected by one or more Area Border Routers (the other main  link state protocol, IS-IS, connects areas somewhat differently) which maintain a separate link state database and calculate a separate shortest-path tree for each of their connected areas. So an ABR by definition is a member of two or more areas. It advertises the prefixes it learns in one area to its other areas by flooding Type 3 LSAs into the areas that basically say, “I know how to reach these destinations.”
Wait a minute – what that last concept described is not link state, it’s distance vector. The routers in an area cannot “see” past the ABR, and  rely on the ABR to correctly tell them what prefixes it can reach. The SPF calculation within an area derives a shortest-path tree that depicts  all prefixes beyond the ABR as leaf subnets connected to the ABR at some specified cost.

And that leads us to the answer to the question:
Because inter-area OSPF is distance vector, it is vulnerable to routing loops. It avoids loops by mandating a loop-free inter-area topology, in which traffic from one area can only reach another area through area 0. 