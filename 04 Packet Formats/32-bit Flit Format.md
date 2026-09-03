---
id: 32-bit-flit-format
title: "32-bit Flit Format"
aliases: ["Flit format","GNet flit"]
type: packet
status: draft
layers: ["L2"]
tags: ["gnet","gnet/packet","gnet/status/draft","gnet/layer/l2"]
parent: "[[Packet Formats MOC]]"
related: ["[[Direct Link Protocol]]","[[GDP Datagram]]","[[ADR-0007 32-bit Flit Format]]","[[ADR-0008 VCID in Every Flit]]"]
updated: 2026-09-03
---
# 32-bit flit format and notation

> [!info] Knowledge graph
> **Up:** [[Packet Formats MOC]] · **Related:** [[Direct Link Protocol]] · [[GDP Datagram]]

Status: **CURRENT DRAFT**

Every wire flit is exactly 32 bits:

```text
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |VCID |S|                 Carried bits [28:0]                   |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

- **VCID:** 2-bit hop-local Virtual Channel Identifier.
- **S / SOF:** 1 for the first flit of a frame, 0 for continuation flits.
- **Carried bits:** 29 bits available to the framed protocol.

The VCID and SOF are part of the 32-bit flit, not sideband.

## First-flit frame type

On the first flit only (`SOF=1`), the first of the 29 carried bits is the one-bit **Frame Type** discriminator. That leaves 28 protocol-specific bits in the first flit. Continuation flits (`SOF=0`) use all 29 carried bits as protocol data.

Current Frame Type values:

- `0` — GDP data frame.
- `1` — Hello frame.

No additional DLP frame types are currently defined. New services and control functions should normally be built as protocols carried by GDP.

## GDP consequence

GDP deliberately exploits the asymmetric first-flit capacity:

- Flit 1: Frame Type plus exactly 28 bits of immediately useful routing metadata.
- Flit 2: 29 bits of checksum/flow metadata.
- Flits 3–6: two 58-bit addresses, each occupying exactly two 29-bit continuation flits.

See [[GDP Datagram]] for the exact layout.

## Ordering

Fields are transmitted most-significant bit first. Multi-bit values use network bit order. No alignment padding is inserted between logical fields unless the packet format explicitly defines reserved bits.

The physical medium may serialize or encode a flit differently; the logical transfer unit remains exactly 32 bits.
