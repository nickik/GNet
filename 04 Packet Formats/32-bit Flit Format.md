---
id: 32-bit-flit-format
title: "32-bit Flit Format"
aliases: ["Flit format","GNet flit"]
type: packet
status: mixed
layers: ["L2"]
tags: ["gnet","gnet/packet","gnet/status/mixed","gnet/layer/l2"]
parent: "[[Packet Formats MOC]]"
related: ["[[Direct Link Protocol]]","[[GDP Datagram]]","[[ADR-0007 32-bit Flit Format]]"]
updated: 2026-09-02
---
# 32-bit flit format and notation

> [!info] Knowledge graph
> **Up:** [[Packet Formats MOC]] · **Related:** [[Direct Link Protocol]] · [[GDP Datagram]] · [[ADR-0007 32-bit Flit Format]]


Status: **ACCEPTED 32-bit logical flit and network order; DRAFT DLP padding/trailer**

Every wire-format diagram in this directory uses the following ruler:

```text
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    One 32-bit GNet flit                       |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

Bits are numbered from 0 at the most-significant bit to 31 at the least-significant bit. Octet 0 occupies bits 0–7. Multi-octet integers use network byte order: the most-significant octet and most-significant 32-bit word are transmitted first.

## DLP frame packaging

For a DLP payload of **N octets**:

```text
   Flit 0                         DLP header
   Flits 1 .. ceil(N / 4)         payload, in order
   Final flit                     CRC-8 followed by 24 reserved zero bits
```

The last payload flit is padded with zero bits after the Nth payload octet. Payload Length counts only meaningful payload octets, not padding. CRC-8 covers the DLP header flit and every transmitted payload flit, including final zero padding; it does not cover the CRC trailer itself.

Total transmitted DLP flits:

```text
   2 + ceil(N / 4)
```

Preamble, clock/synchronization symbols, idle symbols, and physical request/grant signaling are outside the DLP flit count.

## GDP inside DLP

For a GDP packet with M application/transport payload octets, DLP Payload Length is `20 + M` because the GDP header is five flits (20 octets):

```text
   DLP flit 0                     DLP header
   DLP flit 1                     GDP header flit 0
   DLP flit 2                     GDP source address [63:32]
   DLP flit 3                     GDP source address [31:0]
   DLP flit 4                     GDP destination address [63:32]
   DLP flit 5                     GDP destination address [31:0]
   DLP flit 6 ..                  GDP transport/application payload
   Last DLP payload flit          final bytes plus zero padding if needed
   Final DLP flit                 CRC-8 trailer
```

GDP has no separate length field and does not see the DLP CRC trailer or padding. A router removes the incoming DLP envelope, processes the five GDP header flits, and constructs a new DLP envelope for the next link.
