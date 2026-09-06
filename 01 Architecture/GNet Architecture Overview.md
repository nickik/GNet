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
updated: 2026-09-06
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
                -> 32-bit data flits + hop-local VC metadata
                    -> PHY-specific phits
```

A GNet flit always carries exactly 32 data bits. Baseline VC2 associates a 2-bit hop-local VCID with every flit without consuming any flit data bits. There is no SOF bit. The first flit on an inactive allocated VC begins the DLP segment implicitly.

The PHY decides how a flit and its VC metadata map onto physical transfer units (phits). An inline serial VC2 mapping contains 34 logical link bits per flit, but GNet does not require a universal 34-bit physical bus.

## Layer 1 — media and link control

Native local copper starts with the universal [[Minimum GNet-3 NIC|Minimum GNet-3]] compatibility profile. The normal LAN progression is:

```text
GC3   shared 3 Mbit/s flit data
GS3   switched 3 Mbit/s flit data per port/path
GS10  switched ports negotiating 3 or 10 Mbit/s flit data independently
```

The named data rates describe 32-bit flit data. PHY symbol/line rate additionally carries VC metadata and any line-code overhead.

There is no normal shared GC10 product/profile. [[GNet PHY Profiles]], [[GNet Copper Cabling]], [[GNet Modular Connector]], and [[GNet Link Control Protocol|GLCP]] define the native copper mechanism.

GNet Broadband Access remains the residential shared-access family. GNet Carrier Trunk remains a distinct point-to-point infrastructure/trunk family.

## Layer 2 — Direct Link Protocol

DLP supplies hop-local bounded transfer, VC state, link integrity, and protocol adaptation. It has no global source/destination address or end-to-end session state.

Native GNet receiver flow control is credit based:

> **1 credit = guaranteed downstream receive capacity for one complete 32-bit flit and its associated link metadata.**

Credit accounting is independent of PHY phit width. Infrastructure scheduling permission is a separate GRANT. Couplers arbitrate one shared medium; switches propagate credits/path availability through independent outputs and favor wormhole/cut-through forwarding with small buffers.

## Layer 3 — GDP

GDP is the common routed datagram/package. Its semantic header contains Version, Type, Size Class, Hop Limit, QoS, 64-bit Source, and 64-bit Destination.

The current 20-octet GDP header maps naturally to five 32-bit flits. GDP has no checksum, CRC, flow/session ID, reliability state, receive window, fragmentation state, or option chain. Those functions belong either to DLP/GLCP on a hop or to endpoints above GDP.

Routing uses hierarchical prefixes. A dedicated DEC router is an optimized product, but routing is a protocol capability: any capable GNet host may advertise delegated/reachable prefixes when authorized by routing policy.

## Higher layers

GTS and application protocols own tunnels, reliability, sequencing, end-to-end integrity, sessions, security, service selection, directory use, terminal service, RPC, files, voice, and other application semantics.

## Implementation boundary

PLIO and QDX are implementation mechanisms, not network wire layers. QDX-GNET may accelerate DMA, queues, VC/credit bookkeeping, CRC, parsing, and forwarding while remaining behavior-compatible with this specification.
