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

## OSPFv2 Packet Header

![OSPFv2 Packet Header](packet_header.png)

------

## Multicast Groups

### OSPFv2 Multicast groups
| Multicast Group | Description      |
|-----------------|------------------|
| 224.0.0.5       | All OSPF Routers |
| 224.0.0.6       | All DR Routers   |

### OSPFv3 Multicast groups
| Multicast Group | Description                 |
|-----------------|-----------------------------|
| FF02::5         | OSPFv3 Routers              |
| FF02::6         | OSPFv3 Designated Routers   |

------

## Timers
| Interval   | Broadcast | Non-Broadcast  | Point-to-Point  | Point-to-Multipoint  | Point-to-Multipoint Non-Broadcast |
|------------|-----------|----------------|-----------------|----------------------|-----------------------------------|
| Hello      | 10        | 30             | 10              | 30                   | 30                                |
| Dead       | 40        | 120            | 40              | 90                   | 120                               |
| Wait       |           |                | 40              | 90                   |                                   |
| Retransmit | 5         | 5              | 5               | 5                    | 5                                 |

------

## OSPFv2 LSA Types

### LSA Type 1
**LSA Name:** Router LSA
**Route Type:** Intra-Area
**CLI:**
- show ip ospf database router
**Description:**
- Each router within the area will flood a Type 1 Router LSA within the area 
- This LSA describes all of a router's interfaces

### LSA Type 2
**LSA Name:** Network LSA
**Route Type:** Intra-Area
**CLI:**
- show ip ospf database network
**Description:**
- Network LSAs are produced by the DR on every multi-access networks.
- The DR represents the multi-access network and all attached routers as a pseudonode, or a single virtual router. 
- The Network LSA represents a pseudonode just as a Router LSA represents a single physical router.
- The Network LSA lists all attached routers including the DR itself.
- Like Router LSAs, the Network LSA is flooded only within the originating area.

### LSA Type 3
**LSA Name:** Network Summary LSA
**Route Type:** Inter-Area
**CLI:**
- show ip ospf database summary
**Description:**
- Network Summary LSAs are originated by the ABRs.
- These LSAs are sent into an area to advertise destinations outside that area.
- These LSAs are the means by which an ABR tells the internal routers of an attached area what destinations the ABR can reach.
- An ABR also advertises the destinations within its attached areas into the backbone with  Network Summary LSAs.
- Default routes external to the area but internal to the OSPF autonomous system are also advertised by this LSA type.
  
### LSA Type 4
**LSA Name:** ASBR Summary LSA
**Route Type:** Inter-Area
**CLI:** 
- show ip ospf database asbr-summary
- show ip ospf border-routers
**Description:**
- These LSAs are also originated by ABRs.
- ASBR Summary LSAs are identical to Network Summary LSAs except that the destination they advertise is an ASBR.
- The destination advertised by an ASBR Summary LSA will always be a host address, because it is a route to a router.

### LSA Type 5
**LSA Name:** External LSA
**Route Type:** External Routes (E1/E2)
**CLI:** 
- show ip ospf database external
- show ip ospf border-routers
**Description:**
- External LSAs are originated by ASBRs.
- These LSAs advertise either a destination external to the OSPF autonomous system, or a default route external to the OSPF autonomous system.
- External LSAs are flooded throughout the OSPF autonomous system.

### LSA Type 7
**LSA Name:** NSSA External LSA
**Route Type:** External Routes (N1/N2)
**CLI:** 
- show ip ospf database external
- show ip ospf border-routers
**Description:**
- NSSA External LSA are originated by ASBRs within Not-So-Stubby Areas (NSSA).
- They are almost identical to an External LSA, but NSSA External LSAs are flooded only within the Not-So-Stubby Area in which it was originated.

------

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

Note:
- If a router is attached to another AS and is also an NSSA ABR, it may originate a both a Type 5 and a Type 7 LSA for the same network. 
- The Type 5 LSA will be flooded to the backbone and the Type 7 will be flooded into the NSSA. If this is the case, the P-bit must be reset (P=0) in the Type 7 LSA so the Type 7 LSA isn’t again translated into a Type 5 LSA by another NSSA ABR.

## Routing Bit
- The routing bit is not a part of the LSA itself. Routing Bit is an internal maintenance bit used by IOS indicating that the route to the destination advertised by this LSA is valid. So when you see "Routing Bit Set on this LSA," it means that the route to this destination is in the routing table.

---

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

---

## OSPF Filtering

### OSPF RIB Filtering
- distribute-list with ACL
- distribute-list with route-map
- Administrative Distance
- RIB filtering does not stop the flooding of LSAs within the area

### Inter Area Routes
- area X range 10.0.0.0 not-advertise
	- Only effective when ABR is translating from Type 1 LSA to a Type 3 LSA (learn from INTRA Area and advertise INTER Area)
- area X filter-list prefix XXX in
	- prevent routes from being advertised TO the specified area
- area X filter-list prefix XXX out
	- prevent routes from being advertised FROM the specified area

### External Routes (LSA Type 5/7)
Effective to filter when going between areas
Has to be applied on the device that is doing the translation into Type 5 LSA
- summary-address 10.0.0.0 255.255.255.0 not-advertise
- summary-address 10.0.0.0 255.255.255.0 nssa-only

---

# Inter-Area OSPF is Distance Vector / OSPF Loop Prevention

Source: https://www.networkworld.com/article/2348778/my-favorite-interview-question.html

Why does OSPF require all traffic between non-backbone areas to pass through a backbone area (area 0)?
Comparing  three fundamental concepts of link state protocols, concepts that even  most OSPF beginners understand, easily derives the answer to the  question. 

The first concept is this:
Every  link state router floods information about itself, its links, and its  neighbors to every other router. From this flooded information each  router builds an identical link state database. Each router then independently runs a shortest-path-first calculation on its database – a local calculation using distributed information – to derive a shortest-path tree. This tree is a sort of map of the shortest path to every other router.
One of the advantages of link state protocols is that the link state database provides a “view” of the entire network, preventing most routing loops. This is in contrast to distance vector  protocols, in which route information is passed hop-by-hop through the network and a calculation is performed at each hop – a distributed  calculation using local information. Each router along a route is dependent on the router before it to perform its calculations correctly and then correctly pass along the results. When a router advertises the prefixes it learns to its neighbors it’s basically  saying, “I know how to reach these destinations.” And because each distance vector router knows only what its neighbors tell it, and has no “view” of the network beyond the neighbors, the protocol is vulnerable to loops.

The second concept is this:
When link state domains grow large, the flooding and the resulting size of the link  state database becomes a scaling problem. The problem is remedied by breaking the routing domain into areas: That first concept is modified so that flooding occurs only within the boundaries of an area, and the resulting link state database contains only information from the routers in the area. This, in turn, means that each router’s calculated shortest-path tree only describes the path to other routers within the area.

The third concept is this:
OSPF  areas are connected by one or more Area Border Routers (the other main  link state protocol, IS-IS, connects areas somewhat differently) which maintain a separate link state database and calculate a separate shortest-path tree for each of their connected areas. So an ABR by definition is a member of two or more areas. It advertises the prefixes it learns in one area to its other areas by flooding Type 3 LSAs into the areas that basically say, “I know how to reach these destinations.”
Wait a minute – what that last concept described is not link state, it’s distance vector. The routers in an area cannot “see” past the ABR, and  rely on the ABR to correctly tell them what prefixes it can reach. The SPF calculation within an area derives a shortest-path tree that depicts  all prefixes beyond the ABR as leaf subnets connected to the ABR at some specified cost.

And that leads us to the answer to the question:
Because inter-area OSPF is distance vector, it is vulnerable to routing loops. It avoids loops by mandating a loop-free inter-area topology, in which traffic from one area can only reach another area through area 0. 



---
