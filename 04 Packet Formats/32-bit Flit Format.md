---
id: 32-bit-flit-format
title: "32-bit Flit Format"
aliases: ["Flit format","GNet flit"]
type: packet
status: mixed
layers: ["L2"]
tags: ["gnet","gnet/packet","gnet/status/mixed","gnet/layer/l2"]
parent: "[[Packet Formats MOC]]"
related: ["[[Direct Link Protocol]]","[[Virtual Channels and VCIDs]]","[[DLP Segment Size Classes]]","[[GDP Datagram]]","[[ADR-0008 VCID in Every Flit]]"]
updated: 2026-09-02
---
# 32-bit flit format and notation

> [!info] Knowledge graph
> **Up:** [[Packet Formats MOC]] · **Related:** [[Direct Link Protocol]] · [[Virtual Channels and VCIDs]] · [[DLP Segment Size Classes]] · [[GDP Datagram]] · [[ADR-0008 VCID in Every Flit]]

Status: **FROZEN 32-bit total width, 4-bit VCID, and 28 carried bits; DRAFT DLP header/trailer details**

Every wire flit is exactly 32 bits:

```text
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |                  Carried bits [27:0]                  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

Bits 0–3 contain the four-bit Virtual Channel Identifier. Bits 4–31 carry exactly 28 bits of DLP header, protocol payload, padding, or trailer data. The VCID is inside the 32-bit flit; it is not an additional tag outside it.

Bits are numbered from 0 at the most-significant bit to 31 at the least-significant bit. Within the 28-bit carried region, protocol bit 0 is placed first. Multi-octet protocol fields use network byte order and are treated as continuous bitstrings: the most-significant octet and most-significant bit are carried first.

Because 28 is not divisible by eight, protocol octets and 32-bit logical fields routinely cross physical flit boundaries. No alignment padding is inserted between flits.

## DLP segment packaging

For a DLP payload of **N octets**:

```text
   Flit 0                         VCID + 28-bit DLP segment header
   Flits 1 .. K                   VCID + consecutive 28-bit payload chunks
   Flit K+1                       VCID + DLP integrity trailer

   K = ceil((8 × N) / 28) = ceil((2 × N) / 7)
```

The final payload chunk is followed by between zero and 27 zero-padding bits. Payload Length counts meaningful payload octets only. Every flit in the segment repeats the same VCID, although a link scheduler may interleave flits from other active VCIDs between them. [[DLP Segment Size Classes]] bounds the meaningful payload to 64, 256, or 1,024 octets according to the DLP Segment Class.

The current trailer draft places CRC-8 in eight of its 28 carried bits. It is a DLP integrity check and is never part of GDP. The precise DLP integrity encoding remains open.

Preamble, clock/synchronization symbols, idle symbols, and physical request/grant signaling are outside the 32-bit flit definition.

## GDP inside DLP

GDP is a 20-octet logical header followed immediately by its payload. DLP slices that continuous bitstream into 28-bit carried regions. For a GDP packet with M payload octets, DLP Payload Length is `20 + M` and the payload-flit count is:

```text
ceil((160 + 8 × M) / 28)
```

The first six payload flits are shown in [[GDP Datagram]]. The sixth contains the final 20 GDP-header bits plus the first eight GDP-payload bits when a payload is present. GDP has no separate length, padding, checksum, CRC, or other integrity field. A router removes and verifies the incoming DLP envelope, processes GDP, and constructs a new DLP envelope and VCID assignment for the next link.
