---
id: gnet-architecture-overview
title: "GNet Architecture Overview"
aliases: ["Architecture overview"]
type: architecture
status: frozen
layers: ["L1","L2","L3","L4","L5","L6","L7"]
tags: ["gnet","gnet/architecture","gnet/status/frozen"]
parent: "[[Architecture MOC]]"
related: ["[[GNet Layer Model]]","[[34-bit Flit Format]]","[[Virtual Channels and VCIDs]]","[[GNet PHY Profiles]]"]
updated: 2026-09-11
---
# GNet architecture overview

Status: **FROZEN layer boundaries; individual encodings retain their own status**

GNet is one routed protocol architecture carried across local copper LANs, residential access, point-to-point trunks, serial/carrier adaptations, and later optical links.

## Encapsulation

```text
Application/service
    -> GTS or application datagram
        -> GDP package
            -> DLP hop transfer
                -> 34-bit physical flits
```

The baseline physical flit is exactly 34 bits: a 2-bit hop-local VCID plus 32 carried bits. There is no SOF bit. The first data flit on an inactive allocated VC begins the DLP segment implicitly.

## Layer 1 — media and link control

Native local copper starts with the universal [[Minimum GNet-3 NIC|Minimum GNet-3]] compatibility profile. The normal LAN progression is:

```text
GC3   shared 3 Mbit/s
GS3   switched 3 Mbit/s per port/path
GS10  switched ports negotiating 3 or 10 Mbit/s independently
```

There is no normal shared GC10 product/profile. [[GNet PHY Profiles]], [[GNet Copper Cabling]], [[GNet Modular Connector]], and [[GNet Link Control Protocol|GLCP]] define the native copper mechanism.

GNET-A remains the residential shared-access family. GNET-P remains a distinct point-to-point infrastructure/trunk family whose 10/25/50 Mbit/s rate names must not be confused with LAN GNet-10.

## Layer 2 — Direct Link Protocol

DLP supplies hop-local bounded transfer, VC state, and protocol adaptation. It has no global source/destination address or end-to-end session state. The current baseline does not define periodic DLP payload-integrity windows.

Native GNet receiver flow control is credit based:

> **1 credit = guaranteed downstream receive capacity for one physical flit.**

Infrastructure scheduling permission is a separate GRANT. Couplers arbitrate one shared medium; switches propagate credits/path availability through independent outputs and favor wormhole/cut-through forwarding with small buffers.

## Layer 3 — GDP

GDP is the common routed datagram/package. Its initial fixed header contains 2-bit Version, 4-bit Type, 4-bit Size Class, one Local/Global Address Form bit, Destination before Source, and a header-only CRC-8. Global addresses are 64 bits each; Local identifiers are 16 bits each.

Global GDP uses an 8-bit Hop Limit and five-flit fixed header. Local GDP uses a 4-bit Hop Limit and two-flit fixed header. GDP CRC-8 protects Version, Type, Size Class, Address Form, Destination, and Source; it excludes Hop Limit, currently Reserved bits, and payload.

GDP therefore provides no payload-integrity guarantee. End-to-end payload integrity and reliability belong to GTS or another protocol above GDP.

Routing uses hierarchical prefixes. A dedicated DEC router is an optimized product, but routing is a protocol capability: any capable GNet host may advertise delegated/reachable prefixes when authorized by routing policy.

## Higher layers

GTS and application protocols own tunnels, reliability, sequencing, end-to-end integrity, sessions, security, service selection, directory use, terminal service, RPC, files, voice, and other application semantics.

## Implementation boundary

PLIO and QDX are implementation mechanisms, not network wire layers. QDX-GNET may accelerate DMA, queues, VC/credit bookkeeping, CRC, parsing, and forwarding while remaining behavior-compatible with this specification.
