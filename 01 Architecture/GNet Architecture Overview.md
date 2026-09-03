---
id: gnet-architecture-overview
title: "GNet Architecture Overview"
aliases: ["Architecture overview"]
type: architecture
status: frozen
layers: ["L1","L2","L3","L4","L5","L6","L7"]
tags: ["gnet","gnet/architecture","gnet/status/frozen"]
parent: "[[Architecture MOC]]"
related: ["[[GNet Layer Model]]","[[32-bit Flit Format]]","[[Virtual Channels and VCIDs]]","[[GNet PHY Profiles]]"]
updated: 2026-09-03
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
                -> 32-bit physical flits
```

The baseline physical flit is exactly 32 bits: a 2-bit hop-local VCID plus 30 carried bits. There is no SOF bit. The first data flit on an inactive allocated VC begins the DLP segment implicitly.

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

DLP supplies hop-local bounded transfer, VC state, link integrity, and protocol adaptation. It has no global source/destination address or end-to-end session state.

Native GNet receiver flow control is credit based:

> **1 credit = guaranteed downstream receive capacity for one physical flit.**

Infrastructure scheduling permission is a separate GRANT. Couplers arbitrate one shared medium; switches propagate credits/path availability through independent outputs and favor wormhole/cut-through forwarding with small buffers.

## Layer 3 — GDP

GDP is the common routed datagram/package. Its semantic header contains Version, Type, Size Class, Hop Limit, QoS, 64-bit Source, and 64-bit Destination.

GDP has no checksum, CRC, flow/session ID, reliability state, receive window, fragmentation state, or option chain. Those functions belong either to DLP/GLCP on a hop or to endpoints above GDP.

Routing uses hierarchical prefixes. A dedicated DEC router is an optimized product, but routing is a protocol capability: any capable GNet host may advertise delegated/reachable prefixes when authorized by routing policy.

## Higher layers

GTS and application protocols own tunnels, reliability, sequencing, end-to-end integrity, sessions, security, service selection, directory use, terminal service, RPC, files, voice, and other application semantics.

## Implementation boundary

PLIO and QDX are implementation mechanisms, not network wire layers. QDX-GNET may accelerate DMA, queues, VC/credit bookkeeping, CRC, parsing, and forwarding while remaining behavior-compatible with this specification.
