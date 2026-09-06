---
id: 32-bit-flit-format
title: "32-bit Flit and PHY Phit Model"
aliases: ["32-bit Flit Format","Flit format","GNet flit","Flit/phit model"]
type: packet
status: accepted
layers: ["L1","L2"]
tags: ["gnet","gnet/packet","gnet/status/accepted","gnet/layer/l1","gnet/layer/l2"]
parent: "[[Packet Formats MOC]]"
related: ["[[Direct Link Protocol]]","[[Virtual Channels and VCIDs]]","[[GNet PHY Profiles]]","[[ADR-0017 32-bit Data Flit and PHY Phits]]"]
updated: 2026-09-06
---
# 32-bit flit and PHY phit model

Status: **ACCEPTED baseline**

## Flit

Every GNet data flit carries exactly **32 data bits**:

```text
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                         Data [31:0]                           |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

The 32-bit flit is the DLP data and flow-control unit. Link-local metadata such as the VCID is **not taken out of these 32 data bits**.

## Baseline VC2 association

Baseline GNet links associate a two-bit hop-local VCID with every flit:

```text
VC metadata: [ VCID:2 ]
Flit data:   [                         Data:32                         ]
```

This gives four baseline wire VCs. There is no separate SOF bit.

A VC allocated by link control is inactive until its first data flit arrives. That first flit implicitly starts the DLP segment. Segment completion, ABORT, timeout, or reset returns the VC to inactive/reusable state.

Because VC metadata no longer consumes flit data bits, 32-bit protocol words can map naturally to 32-bit flits. Protocols may still define fields or payloads that are not word-aligned, but there is no generic 30-bit carried-region packing rule.

## Phit

A **phit** is the physical transfer unit of a particular PHY. GNet does not require one phit to equal one flit.

A PHY profile may:

- serialize a flit over several phits;
- use a phit wider than a flit;
- carry VC metadata inline with flit data;
- carry VC metadata through sideband/control signaling;
- use another explicitly standardized mapping.

The mapping MUST preserve the association between each flit and its hop-local VCID.

A simple baseline VC2 inline representation carries:

```text
2 VC bits + 32 data bits = 34 logical link bits per flit
```

This **does not** define a universal 34-bit physical bus or universal 34-bit phit. It is merely the logical amount of link information in that mapping before any additional framing or line coding.

## Future wider VCs

A future negotiated profile MAY define a wider VC identifier, for example VC4 or VC8. Such options are not baseline profiles and MUST NOT change the 32-bit flit data width.

## Ordering

Multi-bit protocol values use network bit order. A physical medium may serialize, stripe, frame, or line-code the flit differently as defined by its PHY profile; the architectural data flit remains exactly 32 bits.
