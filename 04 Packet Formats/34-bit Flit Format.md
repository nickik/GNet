---
id: 34-bit-flit-format
title: "34-bit Flit Format"
aliases: ["Flit format","GNet flit"]
type: packet
status: accepted
layers: ["L2"]
tags: ["gnet","gnet/packet","gnet/status/accepted","gnet/layer/l2"]
parent: "[[Packet Formats MOC]]"
related: ["[[Direct Link Protocol]]","[[Virtual Channels and VCIDs]]","[[ADR-0017 34-bit Flit Format]]","[[ADR-0011 Baseline VC2 Flit Without SOF]]"]
updated: 2026-09-06
---
# 34-bit flit format and notation

Status: **ACCEPTED baseline**

Every baseline GNet wire flit is exactly 34 bits:

```text
   +----+--------------------------------+
   |VCID|       Carried bits (32)        |
   +----+--------------------------------+
```

- **VC:** 2-bit hop-local VCID in the baseline VC2 profile.
- **Carried bits:** 32 bits available to DLP/GDP data.
- **SOF:** there is no separate start-of-frame bit.

A VC allocated by link control is inactive until its first data flit arrives. That first flit implicitly starts the DLP segment. Segment completion, ABORT, timeout, or reset returns the VC to inactive/reusable state.

Fields above DLP are packed as continuous bitstreams across the 32-bit carried regions. Baseline protocol octets and logical 32-bit words align with carried-region boundaries; no higher layer may rely on physical alignment in a future negotiated profile.

## Advanced profile

A future negotiated VC4 profile MAY use:

```text
[ VCID:4 | Carried bits:32 ]
```

but VC4 is not the baseline format and must never be assumed before successful capability negotiation.

## Ordering

Multi-bit protocol values use network bit order. The physical medium may serialize or line-code the logical flit differently; the baseline logical transfer unit remains exactly 34 bits.
