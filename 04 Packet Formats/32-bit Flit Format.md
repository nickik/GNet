---
id: 32-bit-flit-format
title: "32-bit Flit Format"
aliases: ["Flit format","GNet flit"]
type: packet
status: accepted
layers: ["L2"]
tags: ["gnet","gnet/packet","gnet/status/accepted","gnet/layer/l2"]
parent: "[[Packet Formats MOC]]"
related: ["[[Direct Link Protocol]]","[[Virtual Channels and VCIDs]]","[[ADR-0007 32-bit Flit Format]]","[[ADR-0011 Baseline VC2 Flit Without SOF]]"]
updated: 2026-09-03
---
# 32-bit flit format and notation

Status: **ACCEPTED baseline**

Every baseline GNet wire flit is exactly 32 bits:

```text
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |VC |                    Carried bits                           |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

- **VC:** 2-bit hop-local VCID in the baseline VC2 profile.
- **Carried bits:** 30 bits available to DLP/GDP data.
- **SOF:** there is no separate start-of-frame bit.

A VC allocated by link control is inactive until its first data flit arrives. That first flit implicitly starts the DLP segment. Segment completion, ABORT, timeout, or reset returns the VC to inactive/reusable state.

Fields above DLP are packed as continuous bitstreams across the 30-bit carried regions. Protocol octets and logical 32-bit words therefore may cross physical-flit boundaries.

## Advanced profile

A future negotiated VC4 profile MAY use:

```text
[ VCID:4 | Carried bits:28 ]
```

but VC4 is not the baseline format and must never be assumed before successful capability negotiation.

## Ordering

Multi-bit protocol values use network bit order. The physical medium may serialize or line-code the logical flit differently; the logical transfer unit remains exactly 32 bits.
