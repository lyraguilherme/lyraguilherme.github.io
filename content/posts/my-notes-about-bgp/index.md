---
title: 'My notes about BGP'
date: 2026-09-05T09:30:00-03:00
draft: false
tags:
   - CCIE
   - Cisco
showTableOfContents: true
featureAlt: "My notes about BGP"
summary: "A summary of BGP compiled during my CCIE journey, gathering information from RFCs, books, Cisco documentation, blogs, and other sources."
---

# Introduction

This post is a summary of BGP that I compiled during my CCIE journey, gathering information from RFCs, books, Cisco documentation, blogs, and other sources.

> **_IMPORTANT:_** I'm still in the process of rebuilding this page from my original notes, so some information may be missing, and the formatting may not yet be fully refined.

------

# Border Gateway Protocol (BGP)

## Fundamentals

BGP is a **path-vector** routing protocol that exchanges reachability information between autonomous systems. It is an **application-layer** protocol: it runs over TCP port 179 and relies on TCP for sequencing, retransmission, and flow control. It computes Layer 3 forwarding information, but the protocol itself sits above the transport layer.

BGP has no hello protocol of its own. There is nothing equivalent to an OSPF hello or an IS-IS hello that finds a peer on a segment and brings it up automatically. A session exists because it was configured, either as an explicit `neighbor` statement or as a range the router is willing to accept connections from. The one real exception is interface-based peering, where BGP borrows IPv6 Neighbor Discovery to learn the peer address. Dynamic peering covers both cases.

BGP also does not run a shortest-path computation over a topology database the way OSPF or IS-IS does. It selects among received paths using a deterministic tie-break sequence driven mostly by policy, not by metric.

Three properties define it:

| Property | Mechanism | Consequence |
|---|---|---|
| Loop-free across domains | `AS_PATH`: a router rejects any path containing its own ASN | Loop prevention is topology-independent and requires no flooding |
| Policy-first | Attributes are set, matched, and rewritten by local policy at every hop | Path selection is an administrative decision, not a metric computation |
| Incremental and reliable | TCP transport, and only changes are advertised after the initial table exchange | Steady-state overhead is near zero, and convergence is slow by design |

BGP carries roughly one million IPv4 unicast prefixes and 200,000+ IPv6 prefixes in the default-free zone. Its design trades convergence speed for stability at that scale.

## IGP versus BGP

| | IGP (OSPF, IS-IS, EIGRP) | BGP |
|---|---|---|
| Domain | One administrative domain | Between domains, and within one at scale |
| Neighbor discovery | Automatic (hellos, multicast) | Configured, or accepted from a listen range. Interface peering uses IPv6 ND |
| Selection driver | Metric (cost, bandwidth, delay) | Attribute sequence, policy-driven |
| Loop prevention | Full topology database (OSPF, IS-IS), or the DUAL feasibility condition and split horizon (EIGRP) | `AS_PATH`, `ORIGINATOR_ID`, `CLUSTER_LIST` |
| Scale ceiling | Thousands of prefixes | Millions of prefixes |
| Convergence | Sub-second achievable | Seconds to minutes without tuning |
| Transport | Raw IP or its own reliable layer | TCP/179 |

The practical division: the IGP carries infrastructure loopbacks and links and resolves BGP next hops. BGP carries everything else. An iBGP next hop that the IGP cannot resolve is an unusable path, and it is worth checking first when a prefix is in the BGP table but missing from the RIB.

## Autonomous System Numbers (ASNs)

An **autonomous system** is what BGP routes between, and RFC 1930, which is BCP 6 and the current authority, defines one as *"a connected group of one or more IP prefixes run by one or more network operators which has a SINGLE and CLEARLY DEFINED routing policy"*.

Two things in that are worth noticing, because the older definition gets repeated everywhere. An AS is delimited by **prefixes and policy**, not by routers, and it may be **run by more than one operator**. RFC 4271 does carry the earlier wording, "a set of routers under a single technical administration", but it introduces that as the *classic* definition and immediately notes it no longer matches practice.

| Range | Size | Status | Reference |
|---|---|---|---|
| 0 | 2/4-byte | Reserved, must not appear in `AS_PATH` | RFC 7607 |
| 1 – 23455 | 2-byte | Public, RIR-allocated | |
| 23456 | 2-byte | `AS_TRANS`, placeholder for 4-byte ASNs | RFC 6793 |
| 23457 – 64495 | 2-byte | Public, RIR-allocated | |
| 64496 – 64511 | 2-byte | Documentation and examples | RFC 5398 |
| 64512 – 65534 | 2-byte | Private use | RFC 6996 |
| 65535 | 2-byte | Reserved | RFC 7300 |
| 65536 – 65551 | 4-byte | Documentation and examples | RFC 5398 |
| 65552 – 4199999999 | 4-byte | Public, RIR-allocated | |
| 4200000000 – 4294967294 | 4-byte | Private use | RFC 6996 |
| 4294967295 | 4-byte | Reserved | RFC 7300 |

**4-byte ASNs (RFC 6793).** A speaker advertises capability code 65 in its OPEN to signal 4-byte support. Between two NEW speakers, `AS_PATH` (type 2) carries 4-byte values directly. When a NEW speaker must traverse an OLD (2-byte-only) speaker:

- The NEW speaker places `AS_TRANS` (23456) in `AS_PATH` for any ASN that does not fit in 2 bytes.
- The full 4-byte path is carried in parallel in `AS4_PATH` (type 17), an **optional transitive** attribute that the OLD speaker propagates without understanding.
- `AGGREGATOR` (type 7) is shadowed the same way by `AS4_AGGREGATOR` (type 18).
- The receiving NEW speaker reconstructs the real path by merging `AS4_PATH` into the tail of `AS_PATH`.
- `AS_TRANS` is not a real AS. Seeing 23456 in a path means a 2-byte-only speaker is on the path.

**asplain versus asdot.** Once ASNs no longer fit in 2 bytes there is more than one way to write them down. RFC 5396 defines three notations:

| Notation | Rule | AS 65535 | AS 65536 | AS 4200000000 |
|---|---|---|---|---|
| **asplain** | Always a single decimal number | `65535` | `65536` | `4200000000` |
| **asdot+** | Always `high.low` | `0.65535` | `1.0` | `64086.59904` |
| **asdot** | Plain below 65536, `high.low` above | `65535` | `1.0` | `64086.59904` |

The two halves are the ASN divided by 65536 and the remainder, so `1.16` is `(1 × 65536) + 16 = 65552`.

RFC 5396 makes **asplain** the standard notation, and it is the IOS-XE default. `bgp asnotation dot` switches display to asdot:

```
router bgp 65552
 bgp asnotation dot
```

The setting changes **how ASNs are displayed and matched**, not how they are entered. Either form is accepted in configuration regardless of the setting.

**The trap is AS_PATH regular expressions.** In asdot the separator is a literal dot, and a dot is also the regex metacharacter for "any character". An access list written for asplain silently stops working, and one written carelessly for asdot matches far more than intended:

```
ip as-path access-list 1 permit _65552_     ! asplain
ip as-path access-list 2 permit _1\.16_     ! asdot, dot escaped
ip as-path access-list 3 permit _1.16_      ! WRONG: also matches 1216, 1_16, 1x16
```

Changing `bgp asnotation` requires `clear ip bgp *` before existing regexes are recompiled against the new format. That reset requirement is a good reason to decide on a notation early rather than changing it on a live network.

------

# Sessions and Transport

## eBGP versus iBGP

The session type is derived, not configured: if the local ASN and the `remote-as` differ, the session is external (eBGP). If they match, it is internal (iBGP).

| Behavior | eBGP | iBGP |
|---|---|---|
| Peer ASN | Different | Same |
| `AS_PATH` on advertise | Local ASN prepended | Unchanged |
| `NEXT_HOP` on advertise | Set to the local address facing the peer | Unchanged (policy can override) |
| `LOCAL_PREF` | Not sent, stripped on receipt | Mandatory on every UPDATE |
| `MED` | Sent to the directly adjacent AS, not propagated further by default | Propagated within the AS |
| Default IP TTL | 1 | 255 |
| Prefix Re-advertisement | Everything, unless a policy to filter is in place | Never to another iBGP peer (**split-horizon rule**) |
| Best-path preference | Preferred over iBGP (step 8) | |
| `MinRouteAdvertisementInterval` | 30 s *(default)* | 0 s *(default)* |

## The iBGP split-horizon rule

This one rule shapes every internal BGP design:

> **A path learned from an iBGP peer is never advertised to another iBGP peer**

| Path learned from | Advertised to eBGP peers | Advertised to iBGP peers |
|---|---|---|
| eBGP peer | Yes | Yes |
| iBGP peer | Yes | **No** |

**Why the rule exists**: `AS_PATH` is what stops a path circulating forever, and a router prepends its ASN only when the path crosses an AS boundary. Inside an AS nothing is prepended, so a path that went R1 to R2 to R3 to R1 would arrive back at R1 carrying exactly the `AS_PATH` it left with. R1 has no way to recognize it as its own. With no attribute to catch the loop, BGP just blocks the second hop.

**What it costs:** Because a path stops after one internal hop, every iBGP speaker has to hear every path directly from the router that learned it. That is a full mesh of `n(n-1)/2` sessions. Ten routers need 45 sessions, fifty need 1225.

**The failure it produces:** In a three-router AS where R1 has the eBGP session and only R1 to R2 and R2 to R3 sessions exist, R3 never learns the external prefix. R2 receives it from R1 over iBGP and stops there. The symptom is a prefix that is present on R2 and simply absent from R3, with no error anywhere and both sessions Established. Anyone who has built an iBGP topology by connecting routers in a line has hit this.

{{< mermaid >}}
flowchart LR
    EXT["AS 64500<br/>originates 203.0.113.0/24"]
    subgraph AS65001["AS 65001"]
        direction LR
        R1["R1<br/>has the prefix"]
        R2["R2<br/>has the prefix"]
        R3["R3<br/>never learns it"]
    end
    EXT -->|eBGP| R1
    R1 -->|iBGP| R2
    R2 -.->|"split horizon<br/>stops it here"| R3
{{< /mermaid >}}

**How the exceptions restore safety:** Both scaling mechanisms work by putting the missing loop detection back before relaxing the rule:

- **Route reflectors (RFC 4456)** permit a reflector to re-advertise iBGP-learned paths, and add `ORIGINATOR_ID` and `CLUSTER_LIST` so a router can still recognize a path that has come back around to it.
- **Confederations (RFC 5065)** turn the boundaries between Member-ASes into eBGP-like sessions, which makes `AS_PATH` work again through `AS_CONFED_SEQUENCE`.

Neither one "bypasses" the rule. Ordinary iBGP speakers still follow it, and a reflector relaxes it only where a client sits on one end of the advertisement: a path from a client goes to everyone, and a path to a client can come from anywhere. Non-client to non-client is still blocked, which is why a reflector with no clients configured behaves exactly like an ordinary iBGP speaker.

## TTL, Multihop, GTSM

TTL is **decremented** by each forwarding router. It is never incremented.

- **eBGP default TTL 1.** Packets cannot survive a router hop, so the peer must be directly connected. This is a reachability constraint, not a policy or advertisement constraint.
- **iBGP default TTL 255.** The opposite assumption. An iBGP peer may be directly connected, but it can equally sit any number of hops away, because the session follows the IGP rather than a single link and loopback-to-loopback peering is common. Starting at the maximum covers both cases, which is why there is no `ibgp-multihop` command.
- **`neighbor X ebgp-multihop [n]`** raises the sent TTL to `n` *(default 255 when no value given)*, permitting a peer several hops away. Required for loopback-to-loopback eBGP and for peering across an intermediate device. It exists only for eBGP, since iBGP already starts at 255.
- **GTSM, RFC 5082** (`neighbor X ttl-security hops N`) inverts the check: send with TTL 255 and **accept only if the received TTL is ≥ 255 − N**. An off-path attacker cannot forge a packet arriving with a high enough TTL, so this cheaply defeats remote spoofing of the session. It is mutually exclusive with `ebgp-multihop` on IOS, so configure one or the other.

```
! Directly connected eBGP, hardened
router bgp 65001
 neighbor 198.51.100.2 remote-as 64500
 neighbor 198.51.100.2 ttl-security hops 1

! Loopback-to-loopback eBGP, two hops away
router bgp 65001
 neighbor 192.0.2.9 remote-as 64501
 neighbor 192.0.2.9 ebgp-multihop 2
 neighbor 192.0.2.9 update-source Loopback0
```

`update-source` sets the source address of the TCP connection. The peer's `neighbor` statement must name exactly that address, or the incoming connection is rejected as unconfigured.

## iBGP and loopbacks

With iBGP sessions, a good practice is to use loopback-to-loopback peering so that the session survives the failure of any single link, provided the IGP has an alternate path. This requires:

- The loopback to be advertised by the IGP.
- `update-source Loopback0` on both ends.
- No `ebgp-multihop` (iBGP already defaults to TTL 255).

## Dynamic peering

The plain form of BGP needs one `neighbor` statement per session, which does not scale when a router aggregates hundreds of peers or when "client" addresses are not known in advance. Two mechanisms can help with that:

**Listen ranges** let a router accept inbound connections from any address inside a prefix and bind each one to a peer group that supplies the policy and the remote ASN:

```
router bgp 65001
 neighbor BRANCHES peer-group
 neighbor BRANCHES remote-as 65100
 neighbor BRANCHES password <secret>
 bgp listen range 10.1.1.0/24 peer-group BRANCHES
 bgp listen limit 200
```

The important detail is that **this is not discovery**. A router with a listen range is **passive on that range**. It never initiates a connection to an address it has not been told about, it only accepts one. The other end still needs a normal `neighbor` statement and still does the initiating. So the many small sites are configured explicitly toward the hub, and the hub listens, which collapses the hub configuration to one block regardless of how many sites come and go.

Points to remember:

- `bgp listen limit` caps the number of dynamic neighbors. The IOS-XE default is 100, and the range is 1 to 5000. Once the limit is hit the router silently stops accepting new sessions.
- The peer group must already exist and must carry `remote-as`, because a dynamic neighbor has no per-neighbor configuration of its own to inherit from.
- Dynamic neighbors are marked with a `*` in `show bgp ipv4 unicast summary`.
- A dynamic session that goes down is torn down completely rather than retried, since there is no configured neighbor to retry toward. The peer reconnects when it is ready.
- `remote-as external` and `remote-as internal` accept any external or any internal ASN, which avoids pinning the peer group to one number.

**Interface peering** is the case that genuinely does discover the neighbor. Instead of naming a peer address, BGP is pointed at an interface and forms the session to whatever IPv6 link-local address it learns from Router Advertisements on that link. There is no addressing to plan and no `neighbor` address to get wrong. It relies on RFC 8950 to carry IPv4 prefixes with an IPv6 next hop.

**IOS-XE does not implement this.** It exists on FRR, Cumulus and Arista. It is worth knowing only because it is the one case where BGP genuinely discovers a neighbor instead of being told about one.

Both mechanisms remove configuration, not filtering. A listen range accepts connections from anything inside the prefix, so it should always be paired with a password on the peer group and with an infrastructure ACL restricting TCP 179.

## Timers

| Timer | RFC 4271 recommendation | IOS-XE default | Configuration |
|---|---|---|---|
| ConnectRetry | 120 s | 120 s | `neighbor X timers connect <10-3600>` |
| Hold | 90 s | **180 s** | `timers bgp <ka> <hold>` or `neighbor X timers <ka> <hold>` |
| Keepalive | 30 s | **60 s** | as above |
| MinRouteAdvertisementInterval | 30 s eBGP / 5 s iBGP | 30 s eBGP / **0 s** iBGP | `neighbor X advertisement-interval <sec>` |
| MinASOriginationInterval | 15 s | n/a | |

The 180/60 pair is a Cisco default, not the RFC value. Both sides propose a hold time in their OPEN, and **the negotiated hold time is the lower of the two**. Keepalives are then normally sent at **1/3 of the negotiated hold time**, so a 180 second hold means a keepalive every 60 seconds, and the peer has to miss three in a row before the session drops. A proposed hold time of 0 disables keepalives and hold-time expiry entirely. A nonzero value must be at least 3 seconds.

Command argument order is `timers <keepalive> <holdtime>`, keepalive first:

```
router bgp 65001
 neighbor 198.51.100.2 timers 10 30      ! keepalive 10 s, hold 30 s
```

**Do not tune timers for fast failure detection.** Sub-second detection is BFD's job, covered further down. Aggressive BGP timers increase CPU load, risk false positives under control-plane congestion, and still cannot detect faster than a few seconds. Use timers to bound the worst case. Use BFD for speed.

## Connection collision

Both peers may open a TCP connection simultaneously, producing two connections for one session. RFC 4271 section 6.8 resolves it deterministically: compare the two BGP Identifiers, and **the connection initiated by the speaker with the higher BGP Identifier survives**. The other is closed with a NOTIFICATION, Cease subcode 7. This is transparent and needs no configuration, but it explains a transient extra session in `debug` output.

------

# Messages and Wire Encoding

## The common header

Every BGP message begins with a **19-byte** fixed header. There is no version field and no magic number in the header. The version lives in the OPEN message body.

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
+                                                               +
|                                                               |
+                                                               +
|                           Marker                              |
+                                                               +
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Length               |      Type     |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

The marker is 16 of the 19 bytes. Everything else in BGP rides behind it.

| Field | Size | Value |
|---|---|---|
| Marker | 16 bytes | All ones (`0xFF` × 16). Vestigial authentication field, retained for framing and synchronization. |
| Length | 2 bytes | Total message length including this header. Minimum 19, maximum 4096. |
| Type | 1 byte | Message type code below. |

| Type | Message | Reference |
|---|---|---|
| 1 | OPEN | RFC 4271 section 4.2 |
| 2 | UPDATE | RFC 4271 section 4.3 |
| 3 | NOTIFICATION | RFC 4271 section 4.5 |
| 4 | KEEPALIVE | RFC 4271 section 4.4 |
| 5 | ROUTE-REFRESH | RFC 2918 |

A Length outside 19–4096, or a Type outside 1–5, is a Message Header Error and resets the session. The upper bound rises if both ends negotiate extended messages, below.

**Extended messages, RFC 8654.** Capability code 6 raises the maximum to **65535 bytes** for every message type except OPEN and KEEPALIVE, so UPDATE, NOTIFICATION and ROUTE-REFRESH can all be extended while those two stay capped at 4096. The capability is directional: a speaker may send extended messages only to a peer that advertised it, so both ends need to advertise it only if both intend to send them. This matters wherever a single UPDATE carries large attribute sets, notably BGP-LS and heavily communitied VPN routes.

## OPEN

Sent once per connection, immediately after TCP establishment.

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+
|    Version    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     My Autonomous System      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           Hold Time           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                         BGP Identifier                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Opt Parm Len  |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
|             Optional Parameters (variable)                    |
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

The fixed part is only 10 bytes. Everything negotiated, every capability, lives in that variable tail, each one a TLV:

```
 0                   1
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-...
|  Parm. Type   | Parm. Length  |  Parameter Value (variable)
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-...
```

| Field | Size | Notes |
|---|---|---|
| Version | 1 byte | Always 4 |
| My Autonomous System | 2 bytes | `AS_TRANS` (23456) if the real ASN needs 4 bytes |
| Hold Time | 2 bytes | Proposed. Negotiated value is `min(local, remote)` |
| BGP Identifier | 4 bytes | Router ID. Must be nonzero and unique within the AS |
| Optional Parameters Length | 1 byte | |
| Optional Parameters | variable | Type 2 = Capabilities (RFC 5492) |

**Capabilities (RFC 5492)** are carried as one or more Optional Parameters of type 2, each holding a Capability Code, length, and value. Codes worth knowing:

| Code | Capability | Reference |
|---|---|---|
| 1 | Multiprotocol Extensions (AFI/SAFI) | RFC 4760 |
| 2 | Route Refresh | RFC 2918 |
| 3 | Outbound Route Filtering (ORF) | RFC 5291 |
| 5 | Extended Next Hop Encoding | RFC 8950 |
| 6 | BGP Extended Message | RFC 8654 |
| 7 | BGPsec | RFC 8205 |
| 9 | BGP Role | RFC 9234 |
| 64 | Graceful Restart | RFC 4724 |
| 65 | 4-byte AS number | RFC 6793 |
| 69 | ADD-PATH | RFC 7911 |
| 70 | Enhanced Route Refresh | RFC 7313 |
| 71 | Long-Lived Graceful Restart | RFC 9494 |

Capabilities are **not negotiated symmetrically by the protocol**. Each side simply announces what it supports. The sender of a feature is responsible for using it only when the peer announced support. A speaker that receives a capability it does not understand must ignore it, not reject the OPEN. Rejecting with OPEN Message Error subcode 7 (Unsupported Capability) is permitted only for capabilities the speaker requires.

If a peer announces no Multiprotocol capability at all, IPv4 unicast is assumed by default.

## UPDATE

The only message that carries routing information. One UPDATE carries **one set of path attributes** and therefore all its announced NLRI share those attributes. Withdrawals need no attributes and can be batched arbitrarily.

Every field is variable length, so RFC 4271 draws this as a stack rather than a bit layout:

```
+-----------------------------------------------------+
|   Withdrawn Routes Length (2 octets)                |
+-----------------------------------------------------+
|   Withdrawn Routes (variable)                       |
+-----------------------------------------------------+
|   Total Path Attribute Length (2 octets)            |
+-----------------------------------------------------+
|   Path Attributes (variable)                        |
+-----------------------------------------------------+
|   Network Layer Reachability Information (variable) |
+-----------------------------------------------------+
```

Withdrawals come first on the wire, which is why a receiver processes them before announcements. Both the withdrawn list and the NLRI use the same encoding:

```
+---------------------------+
|   Length (1 octet)        |
+---------------------------+
|   Prefix (variable)       |
+---------------------------+
```

| Field | Size | Notes |
|---|---|---|
| Withdrawn Routes Length | 2 bytes | 0 if nothing is being withdrawn |
| Withdrawn Routes | variable | List of (prefix length, prefix) |
| Total Path Attribute Length | 2 bytes | 0 if the UPDATE carries no path attributes, as in a withdrawal-only message |
| Path Attributes | variable | See the Path Attributes section |
| NLRI | variable | List of (prefix length, prefix) |

**NLRI encoding.** Each prefix is one length byte giving the prefix length **in bits**, followed by the minimum number of bytes needed to hold that many bits, right-padded with zeros. `10.0.0.0/8` encodes as `08 0A` (2 bytes). `192.0.2.0/24` encodes as `18 C0 00 02` (4 bytes). A default route is a single `00` byte. This variable-length packing is why a full table fits in a manageable number of messages.

An UPDATE with a nonzero Withdrawn Routes Length and no NLRI is a pure withdrawal. A completely empty UPDATE is the **End-of-RIB marker** (RFC 4724), signaling that the initial table transfer is complete, though that encoding only applies to IPv4 unicast. Every other address family signals End-of-RIB with an UPDATE carrying only an `MP_UNREACH_NLRI` attribute for that AFI/SAFI and no withdrawn routes in it.

**Processing order** on receipt: apply withdrawals, then apply announcements, then run best-path for every affected prefix. A prefix that appears in both the withdrawal list and the NLRI list of the same UPDATE ends up announced.

## NOTIFICATION

Sent to report an error. **Sending or receiving a NOTIFICATION always terminates the session** and closes the TCP connection. There is no such thing as a non-fatal NOTIFICATION.

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Error code    | Error subcode |   Data (variable)             |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

| Field | Size |
|---|---|
| Error Code | 1 byte |
| Error Subcode | 1 byte |
| Data | variable |

| Code | Meaning | Notable subcodes |
|---|---|---|
| 1 | Message Header Error | 1 Connection Not Synchronized, 2 Bad Message Length, 3 Bad Message Type |
| 2 | OPEN Message Error | 1 Unsupported Version, 2 Bad Peer AS, 3 Bad BGP Identifier, 4 Unsupported Optional Parameter, 6 Unacceptable Hold Time, 7 Unsupported Capability, 11 Role Mismatch (RFC 9234) |
| 3 | UPDATE Message Error | 1 Malformed Attribute List, 2 Unrecognized Well-known Attribute, 3 Missing Well-known Attribute, 4 Attribute Flags Error, 5 Attribute Length Error, 6 Invalid ORIGIN, 8 Invalid NEXT_HOP, 9 Optional Attribute Error, 10 Invalid Network Field, 11 Malformed AS_PATH |
| 4 | Hold Timer Expired | |
| 5 | Finite State Machine Error | |
| 6 | Cease (RFC 4486) | 1 Maximum Prefixes Reached, 2 Administrative Shutdown, 3 Peer De-configured, 4 Administrative Reset, 5 Connection Rejected, 6 Other Configuration Change, 7 Connection Collision Resolution, 8 Out of Resources |

Codes 4 and 5 are frequently misremembered in the other order. Hold Timer Expired is **4** and FSM Error is **5**.

**Shutdown communication (RFC 8203, updated by RFC 9003)** allows a UTF-8 message of up to 255 bytes in the Data field of Cease subcodes 2 and 4, so an administrative shutdown or reset can carry a ticket reference into the peer's logs instead of appearing as an unexplained teardown. Support is implementation-dependent and it is distinct from graceful shutdown, which is a community rather than a message and is covered under Convergence and Resilience.

## KEEPALIVE

A KEEPALIVE is just the 19-byte message header with the Type field set to 4. Nothing follows it, so the whole message is 19 bytes on the wire.

Normally sent at **1/3 of the negotiated hold time**, which RFC 4271 gives as a reasonable value rather than a fixed requirement. With the IOS-XE default 180 second hold that is one every 60 seconds, which gives the session three chances to hear from the peer before the hold timer expires. Receiving any message resets that timer, not just a KEEPALIVE, so a busy session carrying UPDATEs may send very few keepalives.

## ROUTE-REFRESH

Type 5, defined by RFC 2918 (capability 2). Body is AFI (2 bytes), a reserved byte, SAFI (1 byte). It asks the peer to resend its entire Adj-RIB-Out for that address family, which lets an operator re-apply an inbound policy without resetting the session.

**Enhanced Route Refresh, RFC 7313** (capability 70) reuses the reserved byte as a subtype: 0 = normal request, 1 = Beginning of RR (BoRR), 2 = End of RR (EoRR). The demarcation lets the receiver mark its existing routes stale at BoRR and withdraw whatever was not refreshed by EoRR, which correctly removes prefixes the peer has stopped advertising. Plain RFC 2918 refresh has no such boundaries, so a receiver cannot tell where the resent table starts and ends and has no reliable point at which to clean up what was not resent.

------

# Path Attributes

## Encoding

Path attributes are carried inside the UPDATE message, packed one after another into the Path Attributes field, filling exactly the byte count that the Total Path Attribute Length announced. There is no count of how many attributes follow and no separator between them, so each one has to describe its own size.

That is what **TLV** means: Type, Length, Value. Each attribute states what it is, how many bytes of value it carries, and then the value itself. A parser reads the type, reads the length, skips that many bytes, and lands exactly on the start of the next attribute.

This is the mechanism behind the whole optional-attribute system. A speaker that has never heard of `LARGE_COMMUNITY` still knows where it ends, so it can step over it and keep parsing, and if the transitive bit is set, pass it along untouched. Without self-describing lengths, one unrecognized attribute would make the rest of the message unreadable.

BGP's version puts a flags byte in front, so the layout is really flags, type, length, value:

```
 0                   1
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Attr. Flags  |Attr. Type Code|
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

| Field | Size | Contents |
|---|---|---|
| Attribute Flags | 1 byte | See bit table below |
| Attribute Type Code | 1 byte | See the attribute registry below |
| Attribute Length | 1 or 2 bytes | 2 bytes if the Extended Length flag is set |
| Attribute Value | variable | |

| Bit | Mask | Name | Meaning |
|---|---|---|---|
| 0 | `0x80` | Optional | 0 = well-known, 1 = optional |
| 1 | `0x40` | Transitive | 1 = pass on even if unrecognized. Must be 1 for all well-known attributes |
| 2 | `0x20` | Partial | Set by a speaker that propagates an optional transitive attribute it does not recognize |
| 3 | `0x10` | Extended Length | Length field is 2 bytes rather than 1 |
| 4–7 | | Unused | Must be zero |

So a well-known attribute has flags `0x40`, an optional transitive has `0xC0`, and an optional non-transitive has `0x80`.

## The four categories

| Category | Support required? | Must be present? | Unrecognized handling |
|---|---|---|---|
| **Well-known mandatory** | Yes | In every UPDATE containing NLRI | Cannot happen, absence is an error |
| **Well-known discretionary** | Yes | No | Cannot happen |
| **Optional transitive** | No | No | Propagate unchanged, set the Partial bit |
| **Optional non-transitive** | No | No | Discard silently |

The three **well-known mandatory** attributes are `ORIGIN`, `AS_PATH`, and `NEXT_HOP`. Nothing else. An UPDATE carrying NLRI in the base NLRI field that omits any of the three is a Missing Well-known Attribute error.

One qualification. That applies to classic IPv4 unicast carried in the UPDATE's own NLRI field. When the prefixes travel in `MP_REACH_NLRI` instead, the next hop moves inside that attribute and the standalone `NEXT_HOP` attribute is not sent. `ORIGIN` and `AS_PATH` are still there. See MP-BGP and Address Families.

`LOCAL_PREF` is a special case: it is well-known **discretionary** in general, but it is **mandatory on iBGP and confederation-internal sessions** and **must not be sent to a true eBGP peer**.

## Attribute registry

| Code | Attribute | Category | Reference |
|---|---|---|---|
| 1 | `ORIGIN` | Well-known mandatory | RFC 4271 section 5.1.1 |
| 2 | `AS_PATH` | Well-known mandatory | RFC 4271 section 5.1.2 |
| 3 | `NEXT_HOP` | Well-known mandatory | RFC 4271 section 5.1.3 |
| 4 | `MULTI_EXIT_DISC` | Optional non-transitive | RFC 4271 section 5.1.4 |
| 5 | `LOCAL_PREF` | Well-known discretionary | RFC 4271 section 5.1.5 |
| 6 | `ATOMIC_AGGREGATE` | Well-known discretionary | RFC 4271 section 5.1.6 |
| 7 | `AGGREGATOR` | Optional transitive | RFC 4271 section 5.1.7 |
| 8 | `COMMUNITY` | Optional transitive | RFC 1997 |
| 9 | `ORIGINATOR_ID` | Optional non-transitive | RFC 4456 |
| 10 | `CLUSTER_LIST` | Optional non-transitive | RFC 4456 |
| 14 | `MP_REACH_NLRI` | Optional non-transitive | RFC 4760 |
| 15 | `MP_UNREACH_NLRI` | Optional non-transitive | RFC 4760 |
| 16 | `EXTENDED_COMMUNITIES` | Optional transitive | RFC 4360 |
| 17 | `AS4_PATH` | Optional transitive | RFC 6793 |
| 18 | `AS4_AGGREGATOR` | Optional transitive | RFC 6793 |
| 22 | `PMSI_TUNNEL` | Optional transitive | RFC 6514 |
| 25 | IPv6 Address Specific Ext. Community | Optional transitive | RFC 5701 |
| 26 | `AIGP` | Optional non-transitive | RFC 7311 |
| 29 | `BGP-LS Attribute` | Optional non-transitive | RFC 9552 |
| 32 | `LARGE_COMMUNITY` | Optional transitive | RFC 8092 |
| 35 | `OTC` (Only to Customer) | Optional transitive | RFC 9234 |
| 40 | `BGP Prefix-SID` | Optional transitive | RFC 8669 |

Note in particular that `AGGREGATOR` and `COMMUNITY` are **optional transitive**, not well-known, and that `MED` is **optional non-transitive**, not discretionary.

## ORIGIN (1)

One byte describing how the prefix entered BGP:

| Value | Symbol | Meaning |
|---|---|---|
| **0** | `i` | IGP. Injected by a `network` statement or an aggregate |
| **1** | `e` | EGP. Learned from the historical EGP protocol. Obsolete, never seen in practice |
| **2** | `?` | INCOMPLETE. Injected by `redistribute` |

Lower is preferred in best-path selection: IGP beats EGP beats INCOMPLETE. Memorize **0/1/2**, not 1/2/3.

```
route-map SET-ORIGIN permit 10
 set origin igp
```

Rewriting a redistributed route's origin to `igp` is a common way to stop origin from being the deciding tie-break, but it hides how the prefix entered BGP. Prefer fixing the injection method.

## AS_PATH (2)

A sequence of segments, each encoded as segment type (1 byte), segment length (1 byte, a **count of ASNs, not bytes**), then that many 2- or 4-byte ASNs.

| Type | Segment | Ordered? | Counts toward path length |
|---|---|---|---|
| 1 | `AS_SET` | No | **1**, regardless of how many ASNs it holds |
| 2 | `AS_SEQUENCE` | Yes | Number of ASNs |
| 3 | `AS_CONFED_SEQUENCE` | Yes | **0** |
| 4 | `AS_CONFED_SET` | No | **0** |

Confederation segments (RFC 5065) are invisible to path-length comparison and are stripped when the route leaves the confederation.

**Loop prevention.** On receipt over eBGP, a speaker discards any path whose `AS_PATH` already contains its own ASN. `neighbor X allowas-in [n]` (Cisco-specific) relaxes this, permitting up to `n` occurrences, and it is required in hub-and-spoke L3VPN designs that reuse one ASN across many sites. `neighbor X as-override` (Cisco-specific) solves the same problem from the PE side by rewriting occurrences of the customer's ASN with the provider's before advertising.

**Prepending.** Appending your own ASN several times lengthens the path so remote ASes prefer another entrance. It is the bluntest inbound traffic-engineering tool available and is only effective against ASes that reach step 5 of best path without an earlier decision. A transit provider that sets `LOCAL_PREF` on customer routes will ignore any amount of prepending.

```
route-map PREPEND-TO-ISP2 permit 10
 set as-path prepend 65001 65001 65001
!
router bgp 65001
 neighbor 198.51.100.2 route-map PREPEND-TO-ISP2 out
```

Three prepends is a common convention rather than a limit. Beyond it the returns fall off sharply, since any AS that decided on `LOCAL_PREF` never reached the `AS_PATH` comparison, and some networks apply a maximum AS-path length filter. Equally, one prepend can be ignored entirely by an upstream whose policy sets `LOCAL_PREF` on your routes.

`set as-path prepend last-as <n>` repeats the leftmost ASN already present in the path rather than a number you name. Applied inbound on an ordinary eBGP session that is the sending peer's ASN, which is where it is normally useful.

## NEXT_HOP (3)

The IP address to which traffic for the NLRI should be sent.

| Session | Default behavior |
|---|---|
| eBGP to a directly connected peer | Set to the local interface address facing the peer |
| eBGP, third-party on shared media | May be set to a **third** router's address on the same subnet, if that router is also on the segment and reachable by the peer (RFC 4271 section 5.1.3). Avoids a hairpin on an IXP LAN |
| iBGP | **Unchanged.** Whatever arrived is passed on |

The iBGP behavior is what makes `next-hop-self` necessary. A route learned over eBGP arrives with the external peer's address as `NEXT_HOP`. If that is passed unchanged to iBGP peers, every router in the AS must be able to resolve an address that belongs to a foreign network, typically by carrying the eBGP link subnet in the IGP. Rewriting the next hop to the advertising router's own loopback is cleaner:

```
router bgp 65001
 neighbor 192.0.2.2 remote-as 65001
 neighbor 192.0.2.2 update-source Loopback0
 neighbor 192.0.2.2 next-hop-self
```

`next-hop-self` is **policy, not loop prevention**, and it is **not** a requirement of RFC 4271. The alternatives are to carry the eBGP link subnets in the IGP as passive interfaces, or to redistribute connected. Rewriting is preferred because it decouples internal reachability from external addressing and because it makes the next hop a stable loopback rather than a link address that disappears when the link does.

`next-hop-self all` (Cisco-specific) extends the rewrite to reflected routes on a route reflector, which the plain form deliberately does not touch.

**Resolution is mandatory.** A path whose `NEXT_HOP` is not resolvable in the routing table is not a best-path candidate at all. It is marked inaccessible and skipped before step 1 of the decision process.

## MULTI_EXIT_DISC (4)

Four bytes, **optional non-transitive**, lower is preferred. It is a hint to a neighboring AS about which of several entry points into the local AS it should prefer.

Rules that trip people up:

- **Comparison is restricted.** By default MED is compared only between paths whose **immediately preceding ASN (the leftmost ASN in `AS_PATH`) is the same**. Paths from different neighboring ASes skip the MED step entirely.
- **Propagation is one hop.** A router does not send a received MED on to its own eBGP peers. It does propagate MED across iBGP.
- **Missing MED.** RFC 4271 says treat a missing MED as 0, the best possible value. `bgp bestpath med missing-as-worst` (Cisco-specific) inverts this to 4294967295, which is usually what the operator actually wants.
- **Non-determinism.** Because MED is only comparable within groups, the outcome can depend on the order in which paths were received. `bgp deterministic-med` (Cisco-specific) groups paths by neighboring AS before comparing and should be enabled on every router. It is off by default for backward compatibility.
- `bgp always-compare-med` (Cisco-specific) compares MED across all ASes. Enable it consistently AS-wide or not at all, because a partial deployment produces routing loops.

```
router bgp 65001
 bgp deterministic-med
 bgp bestpath med missing-as-worst
!
route-map MED-OUT permit 10
 match ip address prefix-list PRIMARY-SITE
 set metric 50
route-map MED-OUT permit 20
 set metric 200
```

MED is a request, not an instruction. Many providers strip inbound MED at the edge.

## LOCAL_PREF (5)

Four bytes, higher is preferred, **default 100** on IOS-XE. Well-known discretionary. It is the primary tool for steering **outbound** traffic, because it applies uniformly across the AS: set it once at the edge and every iBGP speaker inherits the same preference.

It must never appear on a true eBGP session. A speaker receiving `LOCAL_PREF` from an eBGP peer discards it. On confederation-internal eBGP sessions it **is** carried, which is one of the defining differences between a confederation and a real inter-AS boundary.

```
route-map FROM-PRIMARY-TRANSIT permit 10
 set local-preference 200
!
router bgp 65001
 neighbor 198.51.100.2 route-map FROM-PRIMARY-TRANSIT in
```

`bgp default local-preference <n>` changes the AS-wide default.

## ATOMIC_AGGREGATE (6)

A **zero-length** well-known discretionary attribute. It is set by a speaker that advertises an aggregate whose `AS_PATH` does not include all the ASNs of the component routes, which happens whenever `as-set` is not used.

Its meaning is a restriction on the receiver: **do not deaggregate, and do not make the NLRI more specific**. It carries no information about which routes were aggregated. RFC 4271 is careful about the strength of the two rules here: a receiver **MUST NOT** make the NLRI more specific, but only **SHOULD NOT** remove the attribute when propagating.

## AGGREGATOR (7)

Optional transitive. Carries the ASN (2 or 4 bytes) and BGP Identifier of the router that performed the aggregation, purely for diagnostics. With 4-byte ASNs in a mixed network it is shadowed by `AS4_AGGREGATOR` (18).

## COMMUNITY (8)

Four bytes, optional transitive, conventionally written `ASN:value` with a 2-byte ASN in the upper half. A route may carry many communities. They are the standard mechanism for signaling policy intent between and within ASes.

**Well-known communities:**

| Value | Name | Effect | Reference |
|---|---|---|---|
| 65535:0 (`0xFFFF0000`) | `GRACEFUL_SHUTDOWN` | Depreference this path, the link is going down for maintenance | RFC 8326 |
| 65535:666 (`0xFFFF029A`) | `BLACKHOLE` | Discard traffic to this prefix | RFC 7999 |
| 65535:65281 (`0xFFFFFF01`) | `NO_EXPORT` | Do not advertise outside the AS (or outside the confederation) | RFC 1997 |
| 65535:65282 (`0xFFFFFF02`) | `NO_ADVERTISE` | Do not advertise to **any** peer | RFC 1997 |
| 65535:65283 (`0xFFFFFF03`) | `NO_EXPORT_SUBCONFED` (`local-AS`) | Do not advertise outside the local sub-AS | RFC 1997 |
| 65535:65284 | `NOPEER` | Do not advertise to bilateral peers | RFC 3765 |
| 65535:6 (`0xFFFF0006`) | `LLGR_STALE` | Route retained past graceful restart, treat as least preferred | RFC 9494 |

`NO_EXPORT` is 65281 and `NO_ADVERTISE` is 65282. Getting these the wrong way round is a classic exam error: `NO_EXPORT` is the **weaker** of the two and the lower number.

**Communities are not sent by default on IOS.** Set them all you like, but without this the peer never sees them:

```
router bgp 65001
 neighbor 198.51.100.2 send-community both     ! standard + extended
```

`both` is required if you need extended communities as well. `standard` and `extended` select one. IOS-XR and Junos send communities by default.

## EXTENDED_COMMUNITIES (16)

Eight bytes, optional transitive, structured so the type space is registered rather than conventional. The first byte is the type, optionally followed by a subtype.

- Bit `0x80` of the type byte: IANA-authority.
- Bit `0x40` of the type byte: **set means non-transitive across ASes**.

| Type | Structure | Common subtypes |
|---|---|---|
| `0x00` / `0x40` | 2-byte AS : 4-byte value | `0x02` Route Target, `0x03` Route Origin (SoO) |
| `0x01` / `0x41` | IPv4 address : 2-byte value | `0x02` Route Target, `0x03` Route Origin |
| `0x02` / `0x42` | 4-byte AS : 2-byte value | `0x02` Route Target, `0x03` Route Origin (RFC 5668) |
| `0x03` / `0x43` | Opaque | `0x0c` Color, `0x0d` Encapsulation |

So a Route Target written `65001:100` is type `0x0002`. **Route Target and Route Distinguisher are unrelated objects**. See Route Distinguisher versus Route Target.

## LARGE_COMMUNITY (32)

Twelve bytes: Global Administrator (4) : Local Data Part 1 (4) : Local Data Part 2 (4), written `65001:1:200`. RFC 8092.

Standard communities cannot express a 4-byte ASN in the global part, which broke community-based policy for every operator holding a 4-byte ASN. Large communities fix it and add a second local field, typically used as (function, parameter). Any network with a 4-byte ASN should be using them.

```
ip bgp-community new-format
!
route-map TAG permit 10
 set large-community 4200000001:1:2001
```

## ORIGINATOR_ID (9) and CLUSTER_LIST (10)

Both optional non-transitive, both defined by RFC 4456, both used exclusively for route-reflection loop prevention. See Route reflection.

## AIGP (26)

Optional non-transitive, RFC 7311, carrying an accumulated IGP metric across a set of cooperating ASes so that end-to-end IGP cost can drive path selection through a multi-AS network under one administration. It is inserted into the decision process early (on IOS-XE, immediately after "locally originated" and before `AS_PATH` length), so it overrides path length. Only meaningful within an AIGP administrative domain, and it must be stripped at its boundary.

## Attribute error handling: RFC 7606

RFC 4271 required a session reset for almost any malformed attribute. In a network carrying a million prefixes, one malformed attribute from one peer therefore tore down a session and reconverged the table. RFC 7606 replaces this with four responses, from mildest to most drastic:

| Action | When |
|---|---|
| **Attribute discard** | The attribute is not needed for correctness: `ATOMIC_AGGREGATE`, `AGGREGATOR`, communities, `ORIGINATOR_ID`/`CLUSTER_LIST` on eBGP |
| **Treat-as-withdraw** | The attribute is needed and cannot be repaired: malformed `ORIGIN`, `AS_PATH`, `NEXT_HOP`, `MED`, `LOCAL_PREF`. The NLRI is treated as if withdrawn |
| **AFI/SAFI disable** | Malformed `MP_REACH_NLRI`/`MP_UNREACH_NLRI` that prevents parsing the NLRI at all |
| **Session reset** | Last resort: the attribute list cannot be parsed, so the message boundary is lost |

Treat-as-withdraw is the important one. It is why a modern implementation drops a bad prefix instead of dropping the peer.

Two related rules are easy to misremember:

- **Repeated attributes.** If `MP_REACH_NLRI` or `MP_UNREACH_NLRI` appears more than once in an UPDATE, the session gets a NOTIFICATION with subcode Malformed Attribute List. For **any other** attribute appearing twice, every occurrence after the first is discarded and the UPDATE keeps being processed. Only the multiprotocol pair is fatal.
- **The first-AS check.** RFC 4271 says a speaker **MAY** check that the leftmost ASN in a path received from an external peer equals that peer's ASN. It is optional, not required. RFC 7606 adds that where the check is performed and fails, the route SHOULD be treated as withdrawn rather than resetting the session.

------

# The Finite State Machine (FSM)

Every BGP session moves through six states, specified in RFC 4271 section 8. A session that will not come up is sitting in one of them, and which one narrows the cause down to a handful of possibilities before you check anything else.

{{< mermaid >}}
stateDiagram-v2
    [*] --> Idle
    Idle --> Connect : ManualStart
    Connect --> OpenSent : TCP up, OPEN sent
    Connect --> Active : TCP failed
    Connect --> Idle : error
    Active --> OpenSent : TCP up, OPEN sent
    Active --> Connect : ConnectRetry expires
    Active --> Idle : error
    OpenSent --> OpenConfirm : valid OPEN received
    OpenSent --> Active : TCP dropped
    OpenSent --> Idle : bad OPEN or hold expiry
    OpenConfirm --> Established : KEEPALIVE received
    OpenConfirm --> Idle : NOTIFICATION, hold expiry, TCP loss
    Established --> Established : UPDATE or KEEPALIVE
    Established --> Idle : NOTIFICATION or hold expiry
{{< /mermaid >}}

Connect and Active bounce off each other while TCP keeps failing. The climb from OpenSent up to Established is one way. Most failures land back in Idle, with one notable exception: losing TCP in OpenSent drops back to Active rather than all the way down.

## States

| State | Meaning | TCP |
|---|---|---|
| **Idle** | All incoming connections refused. Resources released | None |
| **Connect** | Waiting for an outbound TCP connection to complete | SYN sent |
| **Active** | The outbound attempt failed, retrying, and listening for an inbound connection | Listening / retrying |
| **OpenSent** | TCP is up, our OPEN has been sent, waiting for the peer's OPEN | Established |
| **OpenConfirm** | The peer's OPEN was accepted and we sent a KEEPALIVE, waiting for theirs | Established |
| **Established** | Session up. UPDATE, KEEPALIVE, ROUTE-REFRESH and NOTIFICATION flow | Established |

## Transitions

| From | Event | Action | To |
|---|---|---|---|
| Idle | ManualStart / AutomaticStart | Init resources, start ConnectRetryTimer, initiate TCP, listen | Connect |
| Idle | Any other event | Ignore | Idle |
| Connect | TCP connection succeeds | Stop ConnectRetryTimer, send OPEN, set hold timer to 4 min | OpenSent |
| Connect | TCP connection fails | Restart ConnectRetryTimer, keep listening | **Active** |
| Connect | ConnectRetryTimer expires | Drop the attempt, restart timer, retry TCP | Connect |
| Connect | Any error | Release resources | Idle |
| Active | TCP connection succeeds (in or out) | Send OPEN, set hold timer to 4 min | **OpenSent** |
| Active | ConnectRetryTimer expires | Restart timer, initiate TCP | Connect |
| Active | Any error | Release resources | Idle |
| OpenSent | Valid OPEN received | Send KEEPALIVE, set negotiated hold and keepalive timers | OpenConfirm |
| OpenSent | Invalid OPEN | Send NOTIFICATION (code 2) | Idle |
| OpenSent | TCP disconnects | Restart ConnectRetryTimer, listen | **Active** |
| OpenSent | Hold timer expires | Send NOTIFICATION (code 4) | Idle |
| OpenConfirm | KEEPALIVE received | Reset hold timer | **Established** |
| OpenConfirm | NOTIFICATION, hold expiry, TCP loss | Release resources | Idle |
| Established | UPDATE / KEEPALIVE / ROUTE-REFRESH received | Process, reset hold timer | Established |
| Established | Hold timer expires | Send NOTIFICATION (code 4) | **Idle** |
| Established | NOTIFICATION received, or any error | Release resources | **Idle** |

Two transitions are commonly got wrong. **Active goes to OpenSent, never to OpenConfirm.** And **Established goes to Idle on failure, never back to OpenConfirm or Active.** Once a session is up, every error path does funnel through Idle. Before it is up, TCP loss in OpenSent is the exception, returning to Active to keep retrying.

## Reading state in the field

| Stuck in | Almost always means |
|---|---|
| **Idle** | Administratively shut, no route to the peer, or the peer address is not configured on your side. Check `shutdown`, then the RIB |
| **Idle (Admin)** | `neighbor X shutdown` is configured |
| **Active** | TCP cannot complete. Wrong `update-source`, ACL or firewall blocking 179, peer has no matching `neighbor` statement, TTL too low, or the peer is unreachable. **Active is the "I keep trying and failing" state** |
| **Connect** (persistently) | Unusual. A TCP SYN is being sent with no answer and no RST. Silent drop in the path |
| **OpenSent** | TCP completed but the peer sends no OPEN. Note that an MD5 mismatch does **not** land here, since the segments are dropped before TCP completes and the session stays in Connect or Active |
| **OpenConfirm** (persistently) | Our OPEN was accepted and we sent a KEEPALIVE, and the peer's KEEPALIVE never arrives. A bad OPEN parameter such as an unacceptable hold time or a required capability would have produced a NOTIFICATION from OpenSent and gone to Idle, so it never reaches this state |

An oscillating session that keeps returning to Idle points at hold timer expiry (check for CPU or link congestion), a `maximum-prefix` limit being tripped, or MTU/PMTUD failure on the TCP path. That last one is distinctive: the session establishes, exchanges small messages fine, then dies when a large UPDATE is sent.

## Optional FSM features

- **DelayOpen**: pause before sending OPEN so the peer's inbound connection can win the collision, reducing pointless resets during simultaneous start.
- **PassiveTcpEstablishment** (`neighbor X transport connection-mode passive`): never initiate, only listen. Useful when only one side has a fixed address.
- **DampPeerOscillations / IdleHoldTimer**: exponentially back off reconnection attempts for a flapping peer. IOS-XE does this implicitly for repeatedly failing sessions.

------

# RIB Model and Best Path Selection

## The three RIBs

| RIB | Contents | How to see it |
|---|---|---|
| **Adj-RIB-In** | Everything received from a peer, before inbound policy | Conceptual in the RFC model. IOS-XE keeps an inspectable copy only with `soft-reconfiguration inbound`. Without it, route refresh asks the peer to resend rather than recovering anything locally. `show ip bgp neighbors X received-routes` |
| **Loc-RIB** | The routes the Decision Process has **selected**, per RFC 4271. Inbound policy and best-path selection both run before this point | `show ip bgp` |
| **Adj-RIB-Out** | Per-peer, post-outbound-policy view of what is advertised | `show ip bgp neighbors X advertised-routes` |

Only the best path from Loc-RIB is offered to the routing table, and only the best path is advertised to peers (unless add-path is in use). This is why BGP hides path diversity by default and why a route reflector can advertise a path that is suboptimal for some of its clients.

{{< mermaid >}}
flowchart LR
    P1["Peer"] -->|UPDATE| AIN["Adj-RIB-In<br/>received-routes"]
    AIN --> POLIN["inbound<br/>policy"]
    POLIN --> BP["Decision Process<br/>best path selection"]
    BP --> LOC["Loc-RIB<br/>show ip bgp"]
    LOC --> FIB["Routing table"]
    LOC --> POLOUT["outbound<br/>policy"]
    POLOUT --> AOUT["Adj-RIB-Out<br/>advertised-routes"]
    AOUT -->|UPDATE| P2["Peer"]
{{< /mermaid >}}

This is also why there are three different show commands. `received-routes` reads the leftmost box and needs `soft-reconfiguration inbound` to exist at all, `routes` shows what survived inbound policy, and `advertised-routes` reads the rightmost box after outbound policy.

## Best path selection

The next hop must be **resolvable in the routing table** before any comparison happens. A path with an inaccessible next hop is not a candidate. This is a precondition, not a step.

Given two candidate paths, compare in order and stop at the first difference:

| # | Criterion | Prefer | Notes |
|---|---|---|---|
| 1 | **WEIGHT** | Highest | Cisco-specific, never advertised. 32768 for locally originated, 0 for learned. Local to one router |
| 2 | **LOCAL_PREF** | Highest | Default 100. AS-wide |
| 3 | **Locally originated** | Local | `network` or `redistribute` beats `aggregate-address` |
| 4 | **AIGP** | Lowest | RFC 7311. Only if present (placement is Cisco-specific) |
| 5 | **AS_PATH length** | Shortest | `AS_SET` counts 1, confed segments count 0. `bgp bestpath as-path ignore` disables |
| 6 | **ORIGIN** | Lowest | IGP (0) < EGP (1) < INCOMPLETE (2) |
| 7 | **MED** | Lowest | Only between paths from the same neighboring AS, unless `always-compare-med` |
| 8 | **Path type** | eBGP > confed-eBGP > iBGP | |
| 9 | **IGP metric to NEXT_HOP** | Lowest | "Hot potato" routing: hand traffic off at the nearest exit |
| 10 | *(multipath check)* | | If multipath is enabled, paths equal to here are installed together, and selection continues to pick one "best" for advertisement |
| 11 | **Oldest path** | Oldest | eBGP paths only. Stability heuristic: do not churn on an equally good newcomer. Skipped if `bgp bestpath compare-routerid` is set |
| 12 | **Router ID** | Lowest | `ORIGINATOR_ID` is substituted when present |
| 13 | **CLUSTER_LIST length** | Shortest | Fewer reflection hops |
| 14 | **Neighbor address** | Lowest | Final deterministic tie-break |

The parts people get wrong:

- **Router ID: lowest wins.** Not highest.
- **Step 11 is a stability heuristic, not a loop-prevention mechanism**, and it applies only to eBGP paths. It is deliberately non-deterministic across reboots, which is why `bgp bestpath compare-routerid` exists for anyone who wants reproducible selection.
- **Step 9 makes BGP prefer the closest exit**, which is usually what the local AS wants and usually not what the remote AS wants. MED is the remote AS's counter-argument, and it is evaluated two steps earlier, so a MED that is comparable always beats a shorter IGP distance.
- Weight is per-router. Setting weight on one border router to steer traffic does nothing on the others.

## Multipath

```
router bgp 65001
 address-family ipv4 unicast
  maximum-paths 4                    ! eBGP paths
  maximum-paths ibgp 4               ! iBGP paths
  bgp bestpath as-path multipath-relax
```

To be installed as multipath, candidates must be identical through step 9 **and** additionally match on:

- Weight, `LOCAL_PREF`, `ORIGIN`, `MED`
- `AS_PATH` **exactly**, including the ASNs themselves, not merely the length
- Same path type (all eBGP or all iBGP)
- Same IGP metric to the next hop

`bgp bestpath as-path multipath-relax` loosens the `AS_PATH` requirement to equal **length**, allowing load sharing across two different upstream ASes. It is what allows load sharing across two providers, and it is unsafe without deliberate thought, because it will balance across paths with different capacity and latency.

One path is still elected "best" and is the only one advertised to peers. To advertise more than one, use add-path.

## ADD-PATH (RFC 7911)

Capability 69. Normally a speaker advertises exactly one path per prefix per peer, which destroys path diversity behind a route reflector and prevents fast reroute. ADD-PATH prepends a 4-byte **Path Identifier** to each NLRI so a peer can advertise several distinct paths for the same prefix, distinguished by that identifier.

The capability is negotiated **per AFI/SAFI and directionally**: send, receive, or both. A speaker that only signals "receive" will accept multiple paths but still advertise one.

```
router bgp 65001
 address-family ipv4 unicast
  bgp additional-paths send receive
  bgp additional-paths select best 3
  neighbor 192.0.2.2 advertise additional-paths best 3
```

The usual deployment is on route reflectors, so clients see the backup path and can pre-program it for BGP PIC. Note that ADD-PATH is a **capability affecting NLRI encoding**, not a path attribute, and it is unrelated to `maximum-paths`, which is a purely local forwarding decision.

------

# Scaling iBGP

## The full mesh and why it does not scale

The split-horizon rule forces a full mesh of `n(n-1)/2` sessions, which is 45 at ten routers and 1225 at fifty. The configuration burden is the visible cost. The real one is per-session state and the sheer number of TCP connections to keep alive, since every speaker maintains a session with every other. Update generation itself is less costly than the session count suggests, because IOS-XE groups peers with identical outbound policy and builds one update per group, but that does nothing to reduce the number of sessions.

Two standard mechanisms relax the rule without reintroducing loops: route reflection and confederations.

## Route reflection (RFC 4456)

A **route reflector** is allowed to re-advertise iBGP-learned paths. Its peers are divided into **clients** and **non-clients**. RFC 4456 puts it the other way round from how people usually say it: an RR **along with its client peers** forms a **cluster**, so a cluster is one reflector and the many clients under it, not one client and its reflectors.

**Reflection rules:**

| Path learned from | Reflected to |
|---|---|
| **eBGP peer** | All clients and all non-clients (ordinary iBGP behavior) |
| **Client** | All other clients, all non-clients, and all eBGP peers |
| **Non-client** | **Clients only** |

{{< mermaid >}}
flowchart TB
    subgraph FROMCLIENT["Path learned from a CLIENT"]
        direction LR
        c1["Client A"] --> rr1["Route reflector"]
        rr1 --> c2["Client B"]
        rr1 --> n1["Non-client"]
        rr1 --> e1["eBGP peer"]
    end
    subgraph FROMNON["Path learned from a NON-CLIENT"]
        direction LR
        n2["Non-client"] --> rr2["Route reflector"]
        rr2 --> c3["Clients"]
        rr2 -.->|blocked| n3["Other non-clients"]
    end
{{< /mermaid >}}

The asymmetry is the whole point. A client-learned path goes everywhere. A non-client-learned path only reaches clients, because sending it to another non-client would be the plain iBGP re-advertisement the rule forbids.

Clients need no configuration and no awareness that they are clients, they run plain iBGP. Only the reflector is configured:

```
router bgp 65001
 neighbor 192.0.2.11 remote-as 65001
 neighbor 192.0.2.11 update-source Loopback0
 address-family ipv4 unicast
  neighbor 192.0.2.11 activate
  neighbor 192.0.2.11 route-reflector-client
```

**Loop prevention** uses two optional non-transitive attributes, both stripped before the route leaves the AS:

- **`ORIGINATOR_ID` (9)**: the router ID of the iBGP speaker that first injected the path into the AS. Set by the first reflector, never overwritten. A router that receives a path whose `ORIGINATOR_ID` equals its own router ID discards it.
- **`CLUSTER_LIST` (10)**: a reflector prepends its `CLUSTER_ID` before reflecting. A reflector that sees its own `CLUSTER_ID` already in the list discards the path.

`CLUSTER_ID` defaults to the reflector's router ID. It only needs to be configured explicitly when several reflectors serve the same cluster:

```
router bgp 65001
 bgp cluster-id 10.0.0.1
```

**Shared versus unique cluster IDs.** Two reflectors serving the same set of clients may share a `CLUSTER_ID` or use their own. Sharing suppresses redundant reflected copies between the reflectors, saving memory but reducing path diversity. Using unique IDs (the more common modern choice) lets each client see both reflectors' selections, which is what add-path and BGP PIC need. Both are valid. What is invalid is expecting a shared ID to give you path diversity.

**A reflector does not modify `NEXT_HOP`, `AS_PATH`, `LOCAL_PREF`, or `MED` when reflecting.** Plain `next-hop-self` deliberately skips reflected routes, so configuring it on client sessions is harmless and simply has no effect on what gets reflected. `next-hop-self all` is the form that does rewrite reflected paths. That is occasionally what you want, but it changes how clients resolve the next hop and can pull traffic through the reflector itself, so it needs to be a deliberate design decision rather than a habit.

**Suboptimal routing is inherent.** The reflector selects one best path using **its own** IGP distances and reflects that. A client on the far side of the network may have had a nearer exit available that it will now never see. Mitigations: place reflectors where their topological view resembles their clients' (or make them non-forwarding, in the topology but not the data path), use add-path, or use `bgp additional-paths select` to reflect a backup as well.

**Hierarchical reflection** works too: a client of one cluster can itself be a reflector for a lower tier. `CLUSTER_LIST` handles loop prevention across levels.

## Confederations (RFC 5065)

A confederation divides one AS into **Member-ASes**, which is the term RFC 5065 uses. Sessions between them behave like eBGP but preserve internal semantics, and the outside world sees only the **AS Confederation Identifier**.

Two pieces of common lore are not in the RFC. It never uses the word "sub-AS", and it places no requirement that Member-AS numbers come from the private range. Using private ASNs is sensible practice because the numbers never leave the confederation, but it is convention, not specification.

```
router bgp 65501                      ! the member AS number
 bgp confederation identifier 65001   ! what the outside world sees
 bgp confederation peers 65502 65503  ! sibling member ASes
 neighbor 192.0.2.20 remote-as 65502  ! confederation-external session
 neighbor 192.0.2.5  remote-as 65501  ! ordinary iBGP inside the member AS
```

| | Confed-external session | True eBGP session |
|---|---|---|
| `AS_PATH` | `AS_CONFED_SEQUENCE` prepended | `AS_SEQUENCE` prepended |
| Path length contribution | **0** | 1 per ASN |
| `NEXT_HOP` | **Preserved** | Rewritten |
| `LOCAL_PREF` | **Preserved** | Stripped |
| `MED` | **Preserved** | Not propagated |
| Best-path rank | Below true eBGP, above iBGP | Highest |

All `AS_CONFED_*` segments are stripped when a path leaves the confederation, so the member-AS numbers never escape. External ASes see `65001` in place of them, followed by whatever external `AS_SEQUENCE` the path had already accumulated before it entered.

Inside a member AS you still need a full mesh, or a route reflector. Confederations and route reflection combine freely, and large networks often use confederations for organizational boundaries and reflectors within each member AS.

## Choosing between them

| | Route reflection | Confederation |
|---|---|---|
| Migration from full mesh | Easy. Configure reflectors, clients unchanged | Disruptive, every router's ASN changes |
| Policy between groups | No AS-boundary semantics, it is all one AS. Per-neighbor route maps still work | Full eBGP-style policy per member AS |
| Multi-vendor / multi-team boundaries | Weak | Strong |
| Operational familiarity | Very high | Low, rarer in the field |
| Path diversity control | Cluster IDs, add-path | Naturally better (confed-eBGP re-advertises) |

Route reflection is the default answer for almost every network. Confederations earn their complexity when distinct operational teams or acquired networks need real policy boundaries inside one public ASN.

## The synchronization rule

**Historical.** The original rule (from the RFC 1771 era) was: *do not advertise a route learned via iBGP to an eBGP peer unless the same prefix is also reachable through the IGP.* Its purpose was to prevent an AS advertising transit for a destination that its own non-BGP-speaking core routers could not forward to, creating a black hole.

It became obsolete once networks stopped redistributing BGP into their IGP and instead ran BGP on every transit router (or used MPLS, which makes the core prefix-unaware). Cisco disabled synchronization by default in IOS 12.2(8)T. Note the direction: the rule constrained **iBGP-to-eBGP** advertisement, not iBGP-to-iBGP.

------

# Policy

## Filtering tools

| Tool | Matches on | Notes |
|---|---|---|
| `ip prefix-list` | Prefix and prefix length | The correct tool for prefix filtering. Supports `ge`/`le` |
| `ip as-path access-list` | `AS_PATH` regex | Applied to the string form of the path |
| `ip community-list` | Communities | Standard (numbered/named, exact values) or expanded (regex) |
| `route-map` | Anything, and the only tool that can **set** | Ordered, first-match-wins, implicit deny at the end |
| `neighbor X filter-list` | Applies an as-path list | |
| `neighbor X prefix-list` | Applies a prefix list | Cannot be combined with `distribute-list` on the same peer/direction |

## Prefix lists

```
ip prefix-list CUSTOMER-A seq 5  permit 203.0.113.0/24
ip prefix-list CUSTOMER-A seq 10 permit 198.51.100.0/22 le 24
ip prefix-list CUSTOMER-A seq 15 deny   0.0.0.0/0 le 32
```

`ge` and `le` constrain the **prefix length** of matched routes, and both are relative to the stated prefix, not to each other:

- `198.51.100.0/22 le 24` matches 198.51.100.0/22 and any more-specific inside it up to /24.
- `0.0.0.0/0 le 32` matches everything.
- `0.0.0.0/0 ge 25` matches everything longer than /24, the usual "reject too-specific" filter.
- Without `ge`/`le`, the match is exact, length included.

Sequence numbers matter: entries are evaluated in order and there is an implicit `deny` at the end.

## AS_PATH regular expressions

The path is matched as a space-separated string of ASNs.

| Token | Meaning |
|---|---|
| `^` | Start of the path |
| `$` | End of the path |
| `_` | Any delimiter: space, start, end, `{`, `}`, `(`, `)`, or comma |
| `.` | Any single character |
| `*` | Zero or more of the preceding |
| `+` | One or more of the preceding |
| `?` | Zero or one of the preceding |
| `[ ]` | Character class |
| `( )` | Grouping |
| `\|` | Alternation |

Idioms worth memorizing:

```
ip as-path access-list 1  permit ^$              ! locally originated only
ip as-path access-list 2  permit ^64500_         ! learned directly from AS 64500
ip as-path access-list 3  permit _64500$         ! originated by AS 64500
ip as-path access-list 4  permit _64500_         ! transited AS 64500 anywhere
ip as-path access-list 5  permit ^64500(_64500)*$ ! only AS 64500 itself, repeated
ip as-path access-list 6  permit ^[0-9]+$        ! exactly one AS hop away
ip as-path access-list 7  deny   ^$              ! everything except our own
ip as-path access-list 7  permit .*
```

`^64500_` and `_64500$` are the pair that get confused. The **leftmost** ASN is the neighboring AS that sent you the path. The **rightmost** is the origin.

## Community lists

```
ip bgp-community new-format                      ! display as ASN:value, not a 32-bit integer

ip community-list standard CUST-ROUTES permit 65001:100
ip community-list standard BOTH        permit 65001:100 65001:200   ! AND, both must be present
ip community-list expanded ANY-100     permit _65001:100_

ip extcommunity-list standard RT-BLUE  permit rt 65001:100
ip large-community-list standard LC-A  permit 4200000001:1:2001
```

A **standard** list with two communities on one line requires **both**. Two separate `permit` lines are an OR. An **expanded** list takes a regex, which is how you match "any community from ASN 65001".

Setting communities is additive or replacing, and the difference bites:

```
route-map TAG permit 10
 set community 65001:100                    ! REPLACES all existing communities
route-map TAG permit 20
 set community 65001:200 additive           ! appends
route-map TAG permit 30
 set community none                         ! strips all
```

To delete selectively:

```
ip community-list standard STRIP permit 65001:999
route-map CLEAN permit 10
 set comm-list STRIP delete
```

## Route maps

First match wins. An entry with no `match` matches everything, a `deny` entry drops the route, and there is an **implicit deny at the end**, so the final entry of any filter chain must be an explicit `permit` if you want the remaining routes through.

```
ip prefix-list PEER-PREFIXES seq 5 permit 203.0.113.0/24

route-map FROM-PEER permit 10
 match ip address prefix-list PEER-PREFIXES
 set local-preference 150
 set community 65001:1000 additive
route-map FROM-PEER deny 20
 match as-path 4
route-map FROM-PEER permit 30
 set local-preference 90
!
router bgp 65001
 address-family ipv4 unicast
  neighbor 198.51.100.2 route-map FROM-PEER in
```

A route map applied inbound changes the Loc-RIB. Applied outbound it changes what a specific peer sees. Changing a route map does not re-evaluate existing routes until a refresh happens, so see Applying policy changes without a reset.

**Continue clauses** (`continue [seq]`) let one route map apply several sets, but they make policy hard to read and are rarely worth it.

## Aggregation

```
router bgp 65001
 address-family ipv4 unicast
  network 203.0.113.0 mask 255.255.255.0
  aggregate-address 203.0.112.0 255.255.252.0 summary-only
```

`aggregate-address` only generates the aggregate if **at least one more-specific component exists in the BGP table**. This is a feature: the aggregate withdraws itself when the last component disappears, rather than black-holing.

| Option | Effect |
|---|---|
| *(none)* | Advertise the aggregate **and** all components |
| `summary-only` | Advertise the aggregate, suppress all components |
| `as-set` | Build `AS_SET` from the components' `AS_PATH`s. Prevents `ATOMIC_AGGREGATE` being set. **Deprecated, see below** |
| `suppress-map <rm>` | Suppress only the components the map permits |
| `advertise-map <rm>` | Build the aggregate's attributes from only the components the map permits |
| `attribute-map <rm>` | Rewrite the aggregate's own attributes |

Without `as-set`, the aggregate carries an empty `AS_PATH` (it appears locally originated) and `ATOMIC_AGGREGATE` is set. Historically the answer to that was `as-set`, which rebuilds the component ASNs into an `AS_SET` segment and restores loop prevention.

**Do not deploy it.** RFC 9774 is a Standards Track document titled "Deprecation of AS_SET and AS_CONFED_SET in BGP", and it says a speaker **MUST NOT** advertise UPDATE messages containing either segment type. It obsoletes RFC 6472 and updates RFC 4271 and RFC 5065. `AS_SET` was always awkward in practice, since any component flap changes the set and re-advertises the aggregate, and it counts as length 1 no matter how many ASNs it holds.

Know `AS_SET` because it appears in the segment type table and in older material. Aggregate other people's address space with explicit filtering rather than by originating one.

To originate a prefix unconditionally, `network` requires an exactly matching route in the RIB (often a static to `Null0`). To originate conditionally on something else being present, use conditional advertisement.

## Conditional advertisement

Advertise a prefix only while another prefix is (or is not) in the BGP table. The canonical use is a backup link that must stay silent until the primary fails:

```
route-map ADVERTISE-BACKUP permit 10
 match ip address prefix-list OUR-PREFIXES
route-map WATCH-PRIMARY permit 10
 match ip address prefix-list PRIMARY-LEARNED
!
router bgp 65001
 address-family ipv4 unicast
  neighbor 198.51.100.6 advertise-map ADVERTISE-BACKUP non-exist-map WATCH-PRIMARY
```

`non-exist-map`: advertise while the watched prefix is **absent**. `exist-map`: advertise while it is **present**. The check runs on a 60-second cycle, so this is a policy tool, not a convergence tool.

Do not confuse `advertise-map` here with the identically named `aggregate-address advertise-map`. They are unrelated features.

## Outbound Route Filtering (RFC 5291)

ORF lets a receiver push its inbound prefix filter **to the sender**, so the sender never transmits routes the receiver would discard. It saves bandwidth, sender CPU, and receiver memory, and it is the right answer whenever a small customer takes a filtered subset of a large table.

```
router bgp 65001
 address-family ipv4 unicast
  neighbor 198.51.100.2 capability orf prefix-list both
  neighbor 198.51.100.2 prefix-list ONLY-WHAT-I-WANT in
```

Then push it with `clear ip bgp 198.51.100.2 in prefix-filter`. Prefix-based ORF is type 64, and only that type is widely implemented.

## Applying policy changes without a reset

Never use `clear ip bgp *`. It resets every session and reconverges the whole table.

| Method | What it does | Cost |
|---|---|---|
| **Route refresh** (`clear ip bgp X in`) | Asks the peer to resend its Adj-RIB-Out. Session stays up | None, this is the default and correct method |
| **Soft reconfiguration inbound** | Stores the unmodified Adj-RIB-In locally so policy can be re-applied without asking the peer | Significant memory, roughly doubles per-peer storage |
| **Outbound soft** (`clear ip bgp X out`) | Re-runs outbound policy and re-advertises. Always available, never needs peer support | Low |
| **Hard reset** (`clear ip bgp X`) | Tears the session down | Full reconvergence, so avoid |

Route refresh is negotiated automatically (capability 2) and is supported by everything current. `soft-reconfiguration inbound` is only needed when the peer does not support refresh, or when you want to inspect what a peer actually sent you before your policy touched it:

```
router bgp 65001
 address-family ipv4 unicast
  neighbor 198.51.100.2 soft-reconfiguration inbound
!
show ip bgp neighbors 198.51.100.2 received-routes   ! requires the above
show ip bgp neighbors 198.51.100.2 routes            ! post-policy, always available
```

## Peer groups, templates, and update groups

**Peer groups** were the original mechanism: peers sharing an outbound policy are configured once and share a single Adj-RIB-Out computation.

```
router bgp 65001
 neighbor CUSTOMERS peer-group
 neighbor CUSTOMERS remote-as 64500
 neighbor CUSTOMERS route-map CUST-IN in
 neighbor 198.51.100.2 peer-group CUSTOMERS
```

**Peer templates** (`template peer-session` / `template peer-policy`) supersede them, separating session parameters from policy and supporting inheritance.

**Update groups** are the modern reality: IOS-XE automatically groups peers with identical outbound policy and builds one update per group regardless of how you configured them. The performance argument for peer groups is therefore gone. They survive as a configuration convenience. Inspect the real grouping with `show ip bgp update-group`. If two peers you expected to share a group do not, some outbound policy differs between them.

## A complete edge policy

The bare minimum for an eBGP session with a transit provider:

```
ip prefix-list OUR-SPACE seq 5 permit 203.0.113.0/24
!
ip prefix-list SANE-INBOUND seq 5  deny 0.0.0.0/0
ip prefix-list SANE-INBOUND seq 10 deny 0.0.0.0/8 le 32
ip prefix-list SANE-INBOUND seq 15 deny 10.0.0.0/8 le 32
ip prefix-list SANE-INBOUND seq 20 deny 100.64.0.0/10 le 32
ip prefix-list SANE-INBOUND seq 25 deny 127.0.0.0/8 le 32
ip prefix-list SANE-INBOUND seq 30 deny 169.254.0.0/16 le 32
ip prefix-list SANE-INBOUND seq 35 deny 172.16.0.0/12 le 32
ip prefix-list SANE-INBOUND seq 40 deny 192.0.2.0/24 le 32
ip prefix-list SANE-INBOUND seq 45 deny 192.168.0.0/16 le 32
ip prefix-list SANE-INBOUND seq 50 deny 224.0.0.0/3 le 32
ip prefix-list SANE-INBOUND seq 55 deny 0.0.0.0/0 ge 25
ip prefix-list SANE-INBOUND seq 60 permit 0.0.0.0/0 le 24
!
ip as-path access-list 10 deny _65001_        ! reject paths containing our own ASN
ip as-path access-list 10 permit .*           ! private-ASN filtering: see note below
!
route-map TRANSIT-IN permit 10
 match ip address prefix-list SANE-INBOUND
 set local-preference 100
 set community 65001:1000 additive
!
route-map TRANSIT-OUT permit 10
 match ip address prefix-list OUR-SPACE
!
router bgp 65001
 address-family ipv4 unicast
  neighbor 198.51.100.2 remote-as 64500
  neighbor 198.51.100.2 send-community both
  neighbor 198.51.100.2 maximum-prefix 1000000 90 restart 15
  neighbor 198.51.100.2 filter-list 10 in
  neighbor 198.51.100.2 route-map TRANSIT-IN in
  neighbor 198.51.100.2 route-map TRANSIT-OUT out
  neighbor 198.51.100.2 ttl-security hops 1
```

Every eBGP session should have an inbound filter **and** an outbound filter. The outbound one is what stops you becoming an accidental transit provider.

**On filtering private ASNs.** It is tempting to write one regular expression for it. Do not trust a short one. The private ranges are 64512 to 65534 and 4200000000 to 4294967294, and an expression like `_6451[2-9]_` covers eight ASNs out of the first range and none of the second. Getting full coverage in a single regex is possible but unreadable and easy to get subtly wrong, which is worse than no filter because it looks like protection. Generate the list, or filter on the prefixes you expect rather than on the ASNs you do not.

------

# MP-BGP and Address Families

## The AFI/SAFI model

RFC 4271 can only carry IPv4 unicast, because `NEXT_HOP` is a 4-byte field and NLRI has no type information. **RFC 4760** adds two attributes that make BGP carry anything:

- **`MP_REACH_NLRI` (14)**: AFI, SAFI, next-hop length, next hop, then the NLRI. Replaces both `NEXT_HOP` and the UPDATE's NLRI field.
- **`MP_UNREACH_NLRI` (15)**: AFI, SAFI, withdrawn NLRI.

Both are optional non-transitive. An UPDATE carrying `MP_REACH_NLRI` still carries `ORIGIN` and `AS_PATH`, but **not** `NEXT_HOP`, since the next hop moved inside `MP_REACH_NLRI`.

| AFI | Family |
|---|---|
| 1 | IPv4 |
| 2 | IPv6 |
| 16388 | BGP-LS |

| SAFI | Meaning | Reference |
|---|---|---|
| 1 | Unicast | RFC 4760 |
| 2 | Multicast | RFC 4760 |
| 5 | MCAST-VPN | RFC 6514 |
| 71 | BGP-LS | RFC 9552 |
| 73 | SR Policy | BGP SR Policy |
| 128 | MPLS-labeled VPN (VPNv4/VPNv6) | RFC 4364 |
| 129 | Multicast VPN | |

Capability 1 announces each supported (AFI, SAFI) pair. If neither peer announces it, IPv4 unicast is assumed.

## Activating families

On IOS-XE a neighbor defined at the top level is **automatically activated for IPv4 unicast** unless `no bgp default ipv4-unicast` is configured. Every other family requires an explicit `activate`. Forgetting it is worth checking first when a session is up but carrying no routes.

```
router bgp 65001
 no bgp default ipv4-unicast              ! recommended: be explicit everywhere
 neighbor 2001:db8::2 remote-as 64500
 neighbor 192.0.2.2 remote-as 65001
 neighbor 192.0.2.2 update-source Loopback0
 !
 address-family ipv4 unicast
  neighbor 192.0.2.2 activate
 exit-address-family
 !
 address-family ipv6 unicast
  neighbor 2001:db8::2 activate
 exit-address-family
 !
 address-family vpnv4
  neighbor 192.0.2.2 activate
  neighbor 192.0.2.2 send-community extended
 exit-address-family
```

Note that `send-community extended` is mandatory for any VPN family: route targets are extended communities, and without it no route is ever imported anywhere.

## IPv6

IPv6 unicast is AFI 2 / SAFI 1 and behaves identically to IPv4 apart from encoding. Two details:

- The `MP_REACH_NLRI` next-hop field for IPv6 may contain **two** addresses: a global unicast address followed by a link-local address (RFC 2545). The link-local one is only valid on a directly connected session.
- BGP over an IPv6 session can carry IPv4 NLRI and vice versa. The two are independent choices.

## IPv4 over IPv6 next hops

**RFC 8950** (obsoletes RFC 5549) lets IPv4 NLRI carry an **IPv6 next hop**, negotiated with the Extended Next Hop Encoding capability (code 5). This removes the need for IPv4 addressing on transit links entirely, since an IPv4 prefix can be reached through a next hop that only has an IPv6 address.

The practical form of this is a session built over IPv6 transport that carries the IPv4 unicast family. The peering address is IPv6, so any IPv4 prefix learned across it arrives with an IPv6 next hop:

```
router bgp 65001
 no bgp default ipv4-unicast
 neighbor 2001:db8:0:1::2 remote-as 64500
 !
 address-family ipv4 unicast
  neighbor 2001:db8:0:1::2 activate
 exit-address-family
 !
 address-family ipv6 unicast
  neighbor 2001:db8:0:1::2 activate
 exit-address-family
```

A speaker must not advertise IPv4 NLRI with an IPv6 next hop unless the receiver advertised capability 5, so the capability has to be present in the direction the routes flow. What happens without it is implementation-dependent, and the family may simply fail to carry anything useful rather than failing loudly. Verify with `show bgp ipv4 unicast neighbors <peer>` and look for the extended next hop capability in the advertised and received lists.

Support on IOS-XE is limited and release-dependent, so check the platform before designing around it. It is the mechanism underneath interface-based peering, which is why that is an FRR and Arista feature rather than a Cisco one.

------

# L3VPN

## Route Distinguisher versus Route Target

These are two different objects that solve two different problems, and they are easy to confuse.

| | Route Distinguisher | Route Target |
|---|---|---|
| **What it is** | An 8-byte value **prepended to the IPv4 prefix** to make a 12-byte VPNv4 NLRI | An **extended community** attached to the route |
| **Problem solved** | Two customers both using 10.0.0.0/8 must be distinguishable in one BGP table | Which VRFs should import this route |
| **Part of** | The NLRI itself | The path attributes |
| **Can a route have several?** | No, exactly one | Yes, any number |
| **Changes best path?** | Yes: different RDs make different NLRI, so both are carried | No |
| **Where configured** | `rd` under the VRF | `route-target import/export` under the VRF address family |

An RD makes a prefix **unique**. An RT makes a prefix **importable**. An RD does not control distribution, and an RT does not create uniqueness.

**RD formats:**

| Type | Structure | Example |
|---|---|---|
| 0 | 2-byte ASN : 4-byte value | `65001:100` |
| 1 | IPv4 address : 2-byte value | `192.0.2.1:100` |
| 2 | 4-byte ASN : 2-byte value | `4200000001:100` |

A common design uses a **unique RD per VRF per PE** (`<PE-loopback>:<vrf-id>`) rather than one RD per VRF network-wide. Because the RD is part of the NLRI, unique-per-PE RDs mean a route reflector sees two distinct NLRI for a multihomed site and reflects both, preserving path diversity for load sharing and fast reroute. A single shared RD collapses them into one NLRI and the reflector picks one.

## Configuration

```
vrf definition BLUE
 rd 192.0.2.1:100
 address-family ipv4
  route-target export 65001:100
  route-target import 65001:100
 exit-address-family
!
interface GigabitEthernet0/1
 vrf forwarding BLUE
 ip address 10.1.1.1 255.255.255.252
!
router bgp 65001
 address-family vpnv4
  neighbor 192.0.2.9 activate
  neighbor 192.0.2.9 send-community extended
 exit-address-family
 !
 address-family ipv4 vrf BLUE
  neighbor 10.1.1.2 remote-as 64510
  neighbor 10.1.1.2 activate
  redistribute connected
 exit-address-family
```

The forwarding path is: CE sends to PE in the VRF, PE looks up in the VRF's RIB, imposes a **VPN label** (identifying the egress VRF or next hop) and a **transport label** (LDP or SR, identifying the egress PE), and forwards. The P routers only ever see the transport label and know nothing about VPN routes. This is why an MPLS core scales.

## Topologies through route targets

Because import and export are independent, RTs express arbitrary topologies:

| Topology | Configuration |
|---|---|
| **Full mesh** | Every site exports and imports the same RT |
| **Hub and spoke** | Spokes export `RT:spoke`, import `RT:hub`. Hub exports `RT:hub`, imports `RT:spoke`. Traffic between spokes must traverse the hub |
| **Extranet** | Site imports its own RT plus the partner's RT |
| **Management VRF** | Every VRF additionally imports `RT:mgmt`, and the management VRF imports every VRF's RT |

A hub-and-spoke hub usually needs two VRFs (one for the spoke-facing import, one for the hub-facing export) or the `allowas-in`/`as-override` treatment, because the hub re-advertises spoke routes and the spokes would otherwise reject them on `AS_PATH` loop detection.

## Inter-VRF leaking

```
vrf definition BLUE
 address-family ipv4
  route-target import 65001:200          ! import selected routes from GREEN
  import map LEAK-FROM-GREEN             ! and filter what gets imported
```

`import map` filters an RT-based import, it does not create one. For leaking without MP-BGP at all (a single-box case), IOS-XE also offers `route-replicate` (Cisco-specific).

## PE-CE protocols and the loop problem

If the CE runs BGP with the same ASN at multiple sites, each site rejects the other's routes because its own ASN is in the path. Two fixes, and they sit on opposite ends of the link:

```
! On the PE, rewrites the customer ASN with the provider's before advertising
router bgp 65001
 address-family ipv4 vrf BLUE
  neighbor 10.1.1.2 as-override

! On the CE, accepts paths that already contain its own ASN
router bgp 64510
 neighbor 10.1.1.1 allowas-in 2
```

`as-override` is the provider-side fix and is invisible to the customer. `allowas-in` is the customer-side fix and requires touching CE configuration, which is why providers usually prefer `as-override`.

OSPF as the PE-CE protocol uses two separate mechanisms that are often confused. The **DN bit** is the loop prevention signal: a PE sets it on LSAs sent toward a CE, and a PE that receives an LSA with the DN bit set will not redistribute it back into BGP. The **domain identifier** does something different, deciding whether a VPN route is rebuilt for the CE as an inter-area route or as an external one. Only the DN bit prevents loops.

`neighbor X local-as <n>` (Cisco-specific) presents a different ASN to one peer, which is the standard tool for migrating a customer after an acquisition without touching their configuration.

------

# Convergence and Resilience

## What actually takes the time

BGP convergence after a failure is the sum of four independent delays:

| Stage | Typical default | How to shorten |
|---|---|---|
| **Failure detection** | Up to 180 s (hold timer) | BFD, or interface-down detection where the failure is local |
| **Best-path recomputation** | Milliseconds per prefix | Nothing to tune, it scales with table size |
| **RIB/FIB update** | Seconds for a full table | Hardware-dependent. PIC pre-programs the backup |
| **Advertisement to peers** | Up to 30 s (`MinRouteAdvertisementInterval`) | `neighbor X advertisement-interval 0` |

Detection dominates. Everything else is a rounding error until you have fixed detection.

## BFD

Bidirectional Forwarding Detection (RFC 5880) provides sub-second liveness detection independent of the routing protocol, using a lightweight hello in the forwarding plane. BGP registers as a client and tears the session down the instant BFD declares the path down.

```
interface GigabitEthernet0/1
 bfd interval 50 min_rx 50 multiplier 3      ! 150 ms (50 * 3)
!
router bgp 65001
 neighbor 198.51.100.2 fall-over bfd
```

This is the correct answer to "make BGP converge faster," not aggressive keepalives. BFD detects in the data plane, so it catches failures that keep the TCP session nominally alive (a one-way fiber break, a transparent device in the middle). Use single-hop BFD for directly connected peers and multihop BFD for loopback-based sessions.

## Next-hop tracking

A BGP path is only usable while its next hop resolves. **Next-hop tracking** watches the RIB for changes affecting BGP next hops and re-runs best path immediately instead of waiting for a scan cycle:

```
router bgp 65001
 bgp nexthop trigger enable
 bgp nexthop trigger delay 1                 ! seconds to dampen IGP churn
 bgp scan-time 60
```

`bgp nexthop route-map <rm>` restricts which routes are allowed to resolve a BGP next hop. The standard hardening is to forbid a default route from resolving next hops, so that losing the IGP route to a PE does not silently leave every VPN prefix pointing at the default:

```
ip prefix-list NO-DEFAULT seq 5 deny 0.0.0.0/0
ip prefix-list NO-DEFAULT seq 10 permit 0.0.0.0/0 le 32
route-map NH-RESTRICT permit 10
 match ip address prefix-list NO-DEFAULT
!
router bgp 65001
 bgp nexthop route-map NH-RESTRICT
```

## Prefix Independent Convergence (PIC)

Ordinary BGP convergence walks every affected prefix and rewrites its forwarding entry, so restoration time scales with table size: a million prefixes is a long walk. **PIC** restructures the FIB with a level of indirection, so that many prefixes share a pointer to a path-list, and a failure requires updating one path-list rather than a million prefixes.

- **PIC Core**: the IGP path to an unchanged BGP next hop fails. Handled by the IGP's own fast reroute plus the shared indirection. Available essentially for free.
- **PIC Edge**: the BGP next hop itself fails. Requires a **pre-installed backup path**, which requires the router to have received one, which requires either add-path from the route reflector or a second eBGP session.

```
router bgp 65001
 address-family ipv4 unicast
  bgp additional-paths install
```

The dependency chain is the point: PIC Edge needs a backup path in the RIB, which needs path diversity, which needs add-path or unique cluster IDs. Enabling `bgp additional-paths install` on a router that only ever receives one path does nothing.

## Graceful restart

**GR (RFC 4724)**, capability 64. A router whose control plane restarts (a supervisor switchover, a process restart) can keep forwarding with its existing FIB while its BGP sessions rebuild. The peer marks the routes **stale** rather than withdrawing them, holds them for the advertised **restart time**, and flushes whatever was not refreshed once the **End-of-RIB** marker arrives.

Requirements and caveats:

- The restarting router must genuinely preserve its forwarding state. Advertising the "forwarding state preserved" flag without doing so black-holes traffic.
- GR helps a **control-plane** restart. It actively **hurts** a real forwarding failure, because it makes the peer hold routes that no longer work. On a link where failure means the far end is gone, GR delays convergence by the restart time.
- **RFC 8538** extends GR to sessions torn down by NOTIFICATION, which RFC 4724 did not cover.

```
router bgp 65001
 bgp graceful-restart restart-time 120
 bgp graceful-restart stalepath-time 360
```

**Long-Lived Graceful Restart (RFC 9494)**, capability 71, extends the retention well beyond the GR timer (hours), marks retained routes with the `LLGR_STALE` community (65535:6), and requires them to be treated as **least preferred** in the decision process, below any route that is not stale. Setting `LOCAL_PREF` to zero is not the general mechanism, it belongs to an optional partial-deployment procedure for advertising toward peers that did not signal LLGR. It is designed for the case where keeping degraded reachability beats having none, notably on IX route server sessions.

## Route flap dampening

RFC 2439. Each withdrawal or attribute change adds a **penalty** (1000 per flap on IOS). The penalty decays exponentially with a **half-life**. Above the **suppress limit** the prefix is withheld from best-path selection. Below the **reuse limit** it returns, and the **maximum suppress time** caps the punishment.

| Parameter | IOS-XE default |
|---|---|
| Half-life | 15 minutes |
| Reuse limit | 750 |
| Suppress limit | 2000 |
| Maximum suppress time | 60 minutes (4 × half-life) |

```
router bgp 65001
 address-family ipv4 unicast
  bgp dampening 15 750 2000 60
```

**Dampening is off by default and should generally stay off.** RIPE-378 recommended against it in 2006 after measurements showed the defaults suppressed stable prefixes for hours because a single path change deep in the Internet propagates as several updates at the edge. RIPE-580 (2013) permits it only with much higher thresholds (suppress 6000, reuse 750, half-life 15 min, max 60 min). Selective dampening of long prefixes with a `route-map` is defensible. Blanket dampening with defaults is not.

Suppression is a local decision. It does not generate a NOTIFICATION and does not affect the session.

## Maximum prefix

```
 neighbor 198.51.100.2 maximum-prefix 1000000 90 restart 15
 neighbor 198.51.100.6 maximum-prefix 100 80 warning-only
```

Arguments: limit, warning threshold as a percentage, then one of `restart <minutes>` (tear down and retry automatically) or `warning-only` (log and keep accepting). With neither, the session goes down and stays down until manually cleared, which is the safest behavior for a customer session and the most disruptive for a transit session.

Exceeding the limit sends a Cease NOTIFICATION with subcode 1. Every eBGP session should have a limit, and a customer session should have a tight one.

## Graceful shutdown

RFC 8326 defines the `GRACEFUL_SHUTDOWN` community (65535:0). Before planned maintenance, tag your advertisements with it. A peer honoring it sets `LOCAL_PREF` to 0 on those paths, so traffic drains onto the alternative **before** the session goes down rather than during the outage.

```
route-map GSHUT permit 10
 set community 65535:0 additive
!
router bgp 65001
 neighbor 198.51.100.2 route-map GSHUT out
```

Both drain and shutdown are supported directly with `neighbor X shutdown graceful <seconds>`.

------

# Security

## Session authentication

| Mechanism | RFC | Notes |
|---|---|---|
| **TCP-MD5** | RFC 2385 | Ubiquitous, weak by modern standards, no key rollover. Still the practical default |
| **TCP-AO** | RFC 5925 / 5926 | Supports key rollover without dropping the session. Sparse implementation |
| **GTSM** | RFC 5082 | Not authentication, but defeats off-path spoofing cheaply. Deploy alongside |
| **IPsec** | | Heavyweight, occasionally used for multihop sessions over untrusted paths |

```
! TCP-MD5
router bgp 65001
 neighbor 198.51.100.2 password 7 <encrypted>

! TCP-AO (IOS-XE)
key chain BGP-AO tcp
 key 1
  send-id 1
  recv-id 1
  cryptographic-algorithm hmac-sha-256
  key-string 0123456789abcdef0123456789abcdef
!
router bgp 65001
 neighbor 198.51.100.2 ao BGP-AO
```

Three details in that key chain are easy to miss and all three break the session if wrong. The `tcp` keyword on `key chain` is what makes it a TCP-AO chain rather than an ordinary one. Every key needs a `send-id` and a `recv-id` in the range 0 to 255, and **your `send-id` must equal the peer's `recv-id`**, which is the usual reason a first attempt fails. The `key-string` is a master key given as 32 or 64 hex digits, not a passphrase.

On the RFCs: **RFC 5925** defines the TCP-AO framework and **RFC 5926** defines the algorithms it requires, HMAC-SHA-1-96 and AES-128-CMAC-96. The `hmac-sha-256` option above is additional IOS-XE platform support rather than something RFC 5926 mandates.

A mismatched MD5 key produces a session stuck below Established with TCP-level errors rather than a BGP NOTIFICATION, because the segments are discarded before BGP sees them. This is a distinctive symptom.

## What authentication does not do

Session authentication proves the peer is who you configured. It says nothing about whether the routes that peer sends are legitimate. Every real BGP incident of the last two decades came from a correctly authenticated peer sending wrong routes. Filtering is the control that matters.

## Filtering

Non-negotiable on every eBGP session:

1. **Inbound prefix filtering.** From customers: an explicit list of exactly what they are authorized to announce. From peers: their prefixes and their customers'. From transit: bogon and length sanity filtering (see A complete edge policy).
2. **Outbound prefix filtering.** Your prefixes and your customers'. This is what prevents you leaking a full table.
3. **`AS_PATH` filtering.** Reject paths containing your own ASN, private ASNs from the public Internet, and (from a customer) anything longer than their expected topology.
4. **`maximum-prefix`** as a backstop for when a filter is wrong.
5. **Bogon filtering** on both prefixes and origin ASNs. Keep it current, because hardcoded bogon lists rot into outages when a range is allocated.

Large filters are not maintained by hand. Providers build them from the prefixes their customers have registered in a routing registry, and rebuild them on a schedule, which is why a newly acquired prefix is not accepted until the registration catches up.

## Route leaks

RFC 7908 defines a route leak as *"the propagation of routing announcement(s) beyond their intended scope"*, meaning an announcement that violates the intended policy of the receiver, the sender, or an AS along the path.

It catalogues six types. **Type 1, "Hairpin Turn with Full Prefix"**, is the one behind most large incidents: a multihomed AS receives a route from one upstream and passes it to another upstream unchanged, so the update reverses direction and the AS becomes an unintended transit path. RFC 7908 notes these are usually accidental. Outbound filtering prevents all of them.

{{< mermaid >}}
flowchart LR
    PA["Provider A"] -->|"full table"| YOU["Your AS"]
    YOU ==>|"correct:<br/>your own prefixes only"| PB["Provider B"]
    YOU -.->|"leak: re-advertising<br/>Provider A's routes"| PB
{{< /mermaid >}}

The dotted arrow is the whole incident. Provider B now believes you are a valid path to everything Provider A knows, so it starts sending you traffic you were never sized to carry.

## BGP roles and OTC (RFC 9234)

RFC 9234 puts the commercial relationship into the protocol so leaks are prevented automatically rather than by remembering to write a filter.

Each eBGP session is configured with a **role**: Provider, Customer, Peer, RS, or RS-Client. The roles are exchanged in the OPEN (capability 9) and **must be complementary**, or the session is rejected. This alone catches misconfiguration.

The **Only to Customer (OTC)** attribute (type 35, optional transitive) then enforces valley-free propagation:

- A route sent to a Customer, Peer, or RS-Client gets OTC set to the sender's ASN if not already present.
- A route received from a Provider, Peer, or RS gets OTC set to the sender's ASN if not already present.
- A route with OTC set **must not** be advertised to a Provider, Peer, or RS.

The result is that a leak is dropped by the receiver even if the leaking AS forgot its filters.

```
router bgp 65001
 neighbor 198.51.100.2 local-role provider
 neighbor 198.51.100.6 local-role customer strict-mode
```

## Route servers

At an IXP, a **route server (RFC 7947)** lets each participant hold one session instead of n. It is not an ordinary BGP speaker:

- It **does not insert its own ASN** into `AS_PATH`. The path a client sees is what the originating participant sent.
- It **does not modify `NEXT_HOP`**, so traffic flows directly between participants over the IXP fabric rather than through the route server.
- It runs **per-client Adj-RIB-Outs (RFC 7948)** so each participant's bilateral policy can be applied independently. A single shared best-path would give every participant the same answer, which is wrong when policies differ.

Because the route server is transparent, participants must still filter. An IX session needs the same inbound filtering as a transit session.

## Operational baseline

RFC 7454 (BCP 194) has the full checklist. The short version:

- Authenticate every session (MD5 at minimum), plus GTSM.
- Filter prefixes in and out on every eBGP session.
- Filter `AS_PATH`: no own ASN, no private ASNs, no unreasonable lengths.
- `maximum-prefix` on every session.
- Configure roles and OTC where the peer supports them.
- Protect the control plane: an ACL or a Control Plane Policing (CoPP) policy restricting TCP/179 to configured peers. Verify it does not rate-limit BGP itself.
- Log neighbor changes (`bgp log-neighbor-changes`) and alert on them.
- Keep your prefix registrations current with your providers, or their filters will drop what you announce.

------

# Troubleshooting

## Reading `show ip bgp summary`

```
Router# show ip bgp summary
BGP router identifier 192.0.2.1, local AS number 65001
BGP table version is 148921, main routing table version 148921
912443 network entries using 226686864 bytes of memory

Neighbor        V     AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
192.0.2.2       4  65001   14028   13991   148921    0    0 1w2d           412
198.51.100.2    4  64500  882014     991   148921    0    0 3d04h       911803
198.51.100.6    4  64501    1204    1198        0    0    0 00:02:11    Active
198.51.100.10   4  64502    9981    9975   148921    0    0 00:00:34 (NoNeg)
```

| Field | Reading it |
|---|---|
| `State/PfxRcd` numeric | Session is Established, and the number is prefixes **accepted after inbound policy** |
| `State/PfxRcd` textual | Session is not Established. `Idle`, `Active`, `Idle (Admin)` |
| `TblVer` 0 while Established | The peer is up but nothing has been exchanged for this family. Usually a missing `activate` |
| `Up/Down` resetting | Session is flapping. Correlate with logs and interface counters |
| `InQ`/`OutQ` persistently nonzero | The router cannot keep up, or TCP is backed up. Check CPU and MTU |
| `MsgSent` climbing with `MsgRcvd` flat | You are talking, they are not. Usually one-way reachability or authentication |

A very high prefix count from a customer, or a sudden jump, is the signature of a leak in progress.

## Reading a single prefix

```
Router# show ip bgp 203.0.113.0/24
BGP routing table entry for 203.0.113.0/24, version 148903
Paths: (2 available, best #1, table default)
  Advertised to update-groups:
     2
  Refresh Epoch 1
  64500 64510
    198.51.100.2 from 198.51.100.2 (198.51.100.2)
      Origin IGP, metric 0, localpref 200, valid, external, best
      Community: 65001:1000
  Refresh Epoch 2
  64501 64502 64510
    192.0.2.3 (metric 20) from 192.0.2.2 (192.0.2.2)
      Origin IGP, metric 0, localpref 100, valid, internal
      Originator: 192.0.2.3, Cluster list: 192.0.2.2
```

| Line | Meaning |
|---|---|
| `64500 64510` | The `AS_PATH`. Leftmost is the neighboring AS, rightmost is the origin |
| `198.51.100.2 from 198.51.100.2 (198.51.100.2)` | `NEXT_HOP` **from** the peer that sent it **(**their router ID**)**. When the first two differ, the next hop was not rewritten |
| `192.0.2.3 (metric 20)` | The IGP metric to reach that next hop. This is best-path step 9 |
| `valid` | The next hop resolved. **Its absence is the whole diagnosis** |
| `internal` / `external` | iBGP or eBGP path |
| `best` | Won the decision process |
| `Originator` / `Cluster list` | The path was reflected. The cluster list shows how many reflectors it passed |

The two failure modes to recognize instantly: no `valid` keyword means the next hop is unresolvable, and no `best` on any path with `valid` present on none means the prefix is in the table but will never be installed.

## Common problems and what to check

| Symptom | Check in this order |
|---|---|
| **Session stuck in Active** | Route to peer address? Correct `update-source` on both ends? Peer has a `neighbor` line for **your** source address? ACL or CoPP permitting 179? TTL sufficient (`ebgp-multihop`)? MD5 key matching? |
| **Session stuck in Idle** | `neighbor X shutdown`? Any route at all to the peer? |
| **Session flaps every ~3 minutes** | Hold timer expiry. Check CPU, interface errors, and MTU on the path |
| **Session dies on large updates only** | Path MTU Discovery (PMTUD) failure. Test with `ping <peer> size 1500 df-bit`, then fix MTU or clamp TCP MSS |
| **Session establishes, then drops during the initial table transfer** | Control Plane Policing dropping BGP. Looks like PMTUD but is rate-driven, not size-driven. Check `show policy-map control-plane` for drops |
| **Established but zero prefixes** | Missing `activate` for the family, inbound filter denying everything, peer has no outbound policy permitting anything, or `bgp default ipv4-unicast` disabled without an explicit activate |
| **Prefix in `show ip bgp` but not in the RIB** | Next hop unresolvable (**check this first**), or a better source (lower administrative distance) owns the prefix, or the path lost best-path selection |
| **Prefix in the RIB but traffic still leaves elsewhere** | Recursion resolving differently than expected. Check the actual FIB entry with `show ip cef <prefix> detail` |
| **iBGP peer does not see an eBGP-learned route** | The advertiser learned it from another iBGP peer (split horizon), so it needs a reflector or full mesh |
| **Reflector client sees no routes** | `route-reflector-client` missing, or the reflector never selected that path as best |
| **Routes accepted but attributes not what you set** | Route map not applied, applied in the wrong direction, or applied but the session was never refreshed |
| **Communities set locally but peer does not see them** | `send-community` missing on IOS |
| **eBGP multipath not installing** | `AS_PATH` differs and `multipath-relax` is not configured, or the paths differ before step 9 |
| **Aggregate not generated** | No component prefix present in the BGP table |

## Control Plane Policing

CoPP filters traffic destined for the router's own control plane, which is exactly where BGP lives. A policy that is too tight breaks BGP in ways that look like something else, which is what makes it a good trap.

Two distinct failures:

- **Blocked outright.** BGP never gets past TCP. The session sits in **Active** with no OPEN ever exchanged, which is indistinguishable from an ACL or firewall drop until you look at the policy counters.
- **Rate-limited.** Far more common and far more confusing. Keepalives are small and infrequent, so the session comes up and looks healthy. The moment a full table starts flowing the rate hits the policer, packets are dropped, TCP stalls, the hold timer expires and the session resets. It then re-establishes and does the same thing again, so you get a loop that never converges.

The rate-limited case is easy to misdiagnose as a path MTU problem, because both present as "fine until the table transfers". They are separable:

| | PMTUD failure | CoPP policer |
|---|---|---|
| Trigger | Packet **size** | Packet **rate** |
| Reproduces with | `ping <peer> size 1500 df-bit` | Nothing you can ping, it needs load |
| Evidence | Ping with DF fails, small ping works | Drop counters climbing in `show policy-map control-plane` |
| Fix | Correct MTU or clamp TCP MSS | Raise the rate for the BGP class |

```
show policy-map control-plane
show policy-map control-plane input class <bgp-class>
```

Watch the **drop** counters, not the transmit counters. A class that is passing traffic and dropping some is still breaking the session.

A CoPP class for BGP should match TCP port 179 in both directions and be scoped to the addresses you actually peer with, so the class serves as a filter as well as a policer:

```
ip access-list extended COPP-BGP
 permit tcp host 198.51.100.2 any eq bgp
 permit tcp host 198.51.100.2 eq bgp any
!
class-map match-all COPP-BGP
 match access-group name COPP-BGP
!
policy-map COPP
 class COPP-BGP
  police 8000000 conform-action transmit exceed-action transmit
!
control-plane
 service-policy input COPP
```

Setting `exceed-action transmit` on the BGP class while policing other classes is a deliberate choice. It keeps the counters visible for diagnosis without letting the policer take a session down. If you do drop on exceed, size the rate for a full table transfer rather than for steady state, because the two differ by orders of magnitude.

## Useful commands

```
show ip bgp summary
show ip bgp neighbors 198.51.100.2
show ip bgp neighbors 198.51.100.2 received-routes    ! needs soft-reconfiguration inbound
show ip bgp neighbors 198.51.100.2 routes             ! post-inbound-policy
show ip bgp neighbors 198.51.100.2 advertised-routes  ! Adj-RIB-Out
show ip bgp neighbors 198.51.100.2 policy             ! all policy applied to this peer
show ip bgp 203.0.113.0/24 bestpath                   ! why this path won
show ip bgp regexp _64500$
show ip bgp community 65001:1000
show ip bgp filter-list 10
show ip bgp update-group
show bgp vpnv4 unicast all summary
show ip cef 203.0.113.0/24 detail
```

`show ip bgp <prefix> bestpath` states in words which step decided the selection, which is faster than comparing attributes by eye.

Debug is a last resort on a production router with a full table. When it is unavoidable, scope it:

```
access-list 20 permit 198.51.100.2
debug ip bgp 198.51.100.2 updates 20 in
```

`debug ip bgp events` and `debug ip bgp ipv4 unicast` unscoped on a DFZ router will melt the control plane. Use `show` first, always.

------

# Quick Reference

## Message types

| Code | Message |
|---|---|
| 1 | OPEN |
| 2 | UPDATE |
| 3 | NOTIFICATION |
| 4 | KEEPALIVE |
| 5 | ROUTE-REFRESH |

Header: Marker 16 bytes, Length 2 bytes, Type 1 byte = **19 bytes**. Max message 4096, or 65535 with RFC 8654 for everything except OPEN and KEEPALIVE.

## Path attributes

| Code | Attribute | Category | Flags |
|---|---|---|---|
| 1 | ORIGIN | WK mandatory | `0x40` |
| 2 | AS_PATH | WK mandatory | `0x40` |
| 3 | NEXT_HOP | WK mandatory | `0x40` |
| 4 | MULTI_EXIT_DISC | Opt non-transitive | `0x80` |
| 5 | LOCAL_PREF | WK discretionary | `0x40` |
| 6 | ATOMIC_AGGREGATE | WK discretionary | `0x40` |
| 7 | AGGREGATOR | Opt transitive | `0xC0` |
| 8 | COMMUNITY | Opt transitive | `0xC0` |
| 9 | ORIGINATOR_ID | Opt non-transitive | `0x80` |
| 10 | CLUSTER_LIST | Opt non-transitive | `0x80` |
| 14 | MP_REACH_NLRI | Opt non-transitive | `0x80` |
| 15 | MP_UNREACH_NLRI | Opt non-transitive | `0x80` |
| 16 | EXTENDED_COMMUNITIES | Opt transitive | `0xC0` |
| 17 | AS4_PATH | Opt transitive | `0xC0` |
| 18 | AS4_AGGREGATOR | Opt transitive | `0xC0` |
| 26 | AIGP | Opt non-transitive | `0x80` |
| 32 | LARGE_COMMUNITY | Opt transitive | `0xC0` |
| 35 | OTC | Opt transitive | `0xC0` |

Flag bits: `0x80` Optional, `0x40` Transitive, `0x20` Partial, `0x10` Extended Length.

## Best path order

```
0.  Next hop must resolve                (precondition, not a step)
1.  Highest WEIGHT                        (Cisco-specific, local to the router)
2.  Highest LOCAL_PREF                    (default 100)
3.  Locally originated
4.  Lowest AIGP                           (if present)
5.  Shortest AS_PATH                      (AS_SET = 1, confed = 0)
6.  Lowest ORIGIN                         (IGP 0 < EGP 1 < INCOMPLETE 2)
7.  Lowest MED                            (same neighboring AS only)
8.  eBGP > confed-eBGP > iBGP
9.  Lowest IGP metric to NEXT_HOP
10. (multipath installed here)
11. Oldest eBGP path
12. Lowest router ID                      (ORIGINATOR_ID substituted)
13. Shortest CLUSTER_LIST
14. Lowest neighbor address
```

## NOTIFICATION codes

| Code | Meaning |
|---|---|
| 1 | Message Header Error |
| 2 | OPEN Message Error |
| 3 | UPDATE Message Error |
| 4 | **Hold Timer Expired** |
| 5 | **Finite State Machine Error** |
| 6 | Cease |

## Well-known communities

| Value | Name |
|---|---|
| 65535:0 | GRACEFUL_SHUTDOWN |
| 65535:666 | BLACKHOLE |
| 65535:65281 | **NO_EXPORT** |
| 65535:65282 | **NO_ADVERTISE** |
| 65535:65283 | NO_EXPORT_SUBCONFED (local-AS) |
| 65535:65284 | NOPEER |
| 65535:6 | LLGR_STALE |

## Defaults

| Item | RFC 4271 | IOS-XE |
|---|---|---|
| Hold time | 90 s | 180 s |
| Keepalive | 30 s | 60 s |
| ConnectRetry | 120 s | 120 s |
| MRAI eBGP | 30 s | 30 s |
| MRAI iBGP | 5 s | 0 s |
| LOCAL_PREF | n/a | 100 |
| Weight (learned / local) | n/a | 0 / 32768 |
| MED | 0 | 0 |
| eBGP TTL | n/a | 1 |
| iBGP TTL | n/a | 255 |
| Dampening half-life | n/a | 15 min |
| Dampening reuse / suppress | n/a | 750 / 2000 |
| Dampening max suppress | n/a | 60 min |

## AFI / SAFI

| AFI | Family | | SAFI | Meaning |
|---|---|---|---|---|
| 1 | IPv4 | | 1 | Unicast |
| 2 | IPv6 | | 2 | Multicast |
| 16388 | BGP-LS | | 5 | MCAST-VPN |
| | | | 71 | BGP-LS |
| | | | 73 | SR Policy |
| | | | 128 | MPLS VPN (VPNv4/VPNv6) |

------

# Cheat Sheet

Facts that are easy to invert, listed with the correct value.

| Detail | Correct |
|---|---|
| BGP header size | **19 bytes**, not 24. Marker 16 + Length 2 + Type 1 |
| Message type codes | 1 OPEN, 2 UPDATE, 3 NOTIFICATION, 4 KEEPALIVE, 5 ROUTE-REFRESH |
| ORIGIN values | **0** IGP, **1** EGP, **2** INCOMPLETE |
| NOTIFICATION codes 4 and 5 | 4 = Hold Timer Expired, 5 = FSM Error |
| Router ID tie-break | **Lowest** wins |
| `NO_EXPORT` vs `NO_ADVERTISE` | NO_EXPORT = 65535:**65281**, NO_ADVERTISE = 65535:**65282** |
| Well-known mandatory attributes | ORIGIN, AS_PATH, **NEXT_HOP**. Not AGGREGATOR, not COMMUNITY |
| COMMUNITY category | Optional **transitive**, not well-known |
| MED category | Optional **non-transitive**, not discretionary |
| eBGP TTL | Sent as **1**. TTL is decremented by routers, never incremented |
| GTSM direction | Send **255**, accept if received TTL is **high**, not low |
| iBGP split horizon | An iBGP-learned path **is** advertised to eBGP peers, it is not advertised to other iBGP peers |
| Synchronization rule | Constrained **iBGP-to-eBGP** advertisement, not iBGP-to-iBGP. Obsolete |
| `next-hop-self` | Policy convenience, **not** a loop-prevention mechanism and not required by RFC 4271 |
| Route reflector cluster ID | RRs in the **same** cluster share an ID. Sharing costs path diversity |
| RR attribute handling | A reflector does **not** change NEXT_HOP, AS_PATH, LOCAL_PREF, or MED |
| Confederation `AS_PATH` | `AS_CONFED_*` segments contribute **0** to path length |
| Confederation attributes | `LOCAL_PREF`, `MED`, and `NEXT_HOP` **are** preserved across confed-eBGP |
| `AS_SET` length | Counts as **1**, regardless of member count |
| MED comparison | Only between paths from the **same** neighboring AS by default |
| Missing MED | Treated as **0** (best) by default. `missing-as-worst` inverts it |
| ADD-PATH | A **capability** changing NLRI encoding (RFC **7911**), not a path attribute |
| ADD-PATH vs multipath | ADD-PATH is about advertising several paths, `maximum-paths` is about installing them locally. Independent |
| Route refresh RFC | **2918**, enhanced by **7313** |
| 4-byte ASN RFC | **6793** (obsoletes 4893). Notation is RFC **5396** |
| asdot regexes | The dot is a regex metacharacter, so it must be escaped: `_1\.16_`, not `_1.16_` |
| L3VPN RFC | **4364**, not 4271 or 4760 |
| RD vs RT | RD makes the NLRI **unique**, RT controls **import**. Neither does the other's job |
| Dampening defaults | Half-life **15 minutes**, max suppress **60 minutes**. Not seconds, not 65 hours |
| Dampening status | Off by default and discouraged (RIPE-378/580) |
| `timers` argument order | `timers <keepalive> <holdtime>`, keepalive first |
| Hold time negotiation | The **lower** of the two proposals wins |
| Collision resolution | The connection from the **higher** BGP Identifier survives |
| Active state | Means "TCP is failing and I am retrying **this** peer". It does not mean trying a different peer |
| FSM from Active | Goes to **OpenSent** on TCP success, never to OpenConfirm |
| FSM from Established | Goes to **Idle** on any error, never back to Active or OpenConfirm |
| Communities on IOS | Not sent unless `neighbor X send-community` is configured |
| VPN families | Require `send-community extended`, or nothing imports |
| Aggregate generation | Requires at least one component in the BGP table |
| `as-set` | Preserves `AS_PATH` information and suppresses `ATOMIC_AGGREGATE`, but originating `AS_SET` is **MUST NOT** per RFC 9774 |
| Fast failure detection | **BFD**, not aggressive keepalives |

------

# References

**Core**
- RFC 4271: BGP-4
- RFC 4760: Multiprotocol extensions (MP-BGP)
- RFC 6793: 4-byte ASN support (obsoletes 4893)
- RFC 5396: Textual representation of AS numbers (asplain, asdot)
- RFC 7606: Revised attribute error handling
- RFC 8654: Extended message size
- RFC 5492: Capabilities advertisement
- RFC 7607: Codifying AS 0
- RFC 6996 / 7300: Private and reserved ASNs

**Attributes and policy**
- RFC 1997: Communities
- RFC 4360 / 5668: Extended communities
- RFC 8092: Large communities
- RFC 7311: AIGP
- RFC 5291: Outbound Route Filtering
- RFC 2918 / 7313: Route refresh, enhanced route refresh
- RFC 7911: ADD-PATH
- RFC 8326: Graceful shutdown community
- RFC 7999: BLACKHOLE community
- RFC 3765: NOPEER community

**Scaling**
- RFC 4456: Route reflection
- RFC 5065: Confederations

**Services**
- RFC 4364: BGP/MPLS IP VPNs (L3VPN)
- RFC 6514: MVPN
- RFC 8950: IPv4 NLRI with IPv6 next hop (obsoletes 5549)
- RFC 2545: BGP-4 multiprotocol extensions for IPv6
- RFC 9552: BGP-LS (obsoletes 7752)

**Resilience**
- RFC 4724: Graceful restart
- RFC 8538: GR for BGP with NOTIFICATION
- RFC 9494: Long-lived graceful restart
- RFC 5880 / 5881: BFD
- RFC 2439: Route flap damping
- RFC 4486: Cease subcodes
- RFC 8203 / 9003: Shutdown communication

**Security**
- RFC 2385: TCP MD5
- RFC 5925 / 5926: TCP-AO and its algorithms
- RFC 5082: GTSM
- RFC 7908: Route leak definitions
- RFC 9234: BGP roles and OTC
- RFC 7454: BGP operations and security (BCP 194)
- RFC 7947 / 7948: Route server operation and model
