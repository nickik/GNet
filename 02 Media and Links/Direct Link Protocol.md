---
id: direct-link-protocol
title: "Direct Link Protocol"
aliases: ["DLP","GNET-LINK"]
type: protocol
status: draft
layers: ["L2"]
tags: ["gnet","gnet/protocol","gnet/status/draft","gnet/layer/l2"]
parent: "[[Media and Links MOC]]"
related: ["[[32-bit Flit Format]]","[[GDP Datagram]]","[[Virtual Channels and VCIDs]]","[[ADR-0004 Direct Point-to-Point Link]]"]
updated: 2026-09-03
---
# Direct Link Protocol (DLP)

> [!info] Knowledge graph
> **Up:** [[Media and Links MOC]] · **Related:** [[32-bit Flit Format]] · [[GDP Datagram]]

Status: **CURRENT DRAFT**

DLP is the minimal Layer-2 protocol for direct links. It deliberately avoids network addressing, sessions, routing semantics, and application/service semantics. Those belong to GDP and protocols above GDP.

## Fundamental flit format

Every transmitted flit is exactly 32 bits:

```text
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |VCID |S|                 Carried bits [28:0]                   |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

- **VCID:** 2-bit hop-local Virtual Channel Identifier.
- **SOF:** 1 for the first flit of a frame, 0 for continuation flits.
- **Carried bits:** 29 bits.

On the first flit only, the first carried bit is the one-bit **Frame Type** discriminator. Thus the first flit has 28 protocol-specific carried bits after Frame Type. Continuation flits have all 29 carried bits available.

## Frame Type

The initial registry is intentionally minimal:

- `0` — GDP data frame.
- `1` — Hello frame.

Anything more complex than link-local Hello should normally be represented as GDP data and implemented above GDP instead of creating additional DLP frame types.

## Hello

Hello is a single-flit, link-local presence frame. It is never routed and carries no source/destination addresses, timing interval, routing table, or session state.

```text
[ VCID:2 | SOF=1 | FrameType=1 | Type:4 | Version:4 | Reserved:12 | Checksum:8 ]
```

The reserved bits transmit as zero. The 8-bit checksum is the final field of the Hello flit.

## GDP carriage

For GDP, Frame Type is `0`. The GDP header occupies six flits. Flit 1 uses the remaining 28 first-flit bits exactly; Flit 2 uses all 29 carried bits; two 58-bit addresses then occupy four complete continuation flits. See [[GDP Datagram]].

## Integrity

DLP uses an **8-bit CRC** as its link-level accidental-error detector. This is intended to detect corruption caused by the physical link, including marginal or overly long serial links, while remaining inexpensive in early hardware.

The exact CRC-8 polynomial, initialization/reflection convention, and final placement in multi-flit DLP frames remain to be frozen. Hello additionally contains its own single-flit 8-bit checksum as specified above. GDP carries a separate lightweight header checksum for its routed header; GNet or another higher layer is responsible for end-to-end payload integrity when required.

## Link semantics

DLP assumes direct adjacency. Local physical identity comes from the actual point-to-point connection, router port, premise channel, or equivalent medium-specific endpoint. DLP therefore has no MAC source or destination address fields.

VCID state is hop-local and is replaced/terminated at forwarding nodes. The link scheduler may interleave flits from different VCIDs according to the medium and flow-control rules.
