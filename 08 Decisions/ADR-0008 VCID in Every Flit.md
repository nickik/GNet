---
id: adr-0008-vcid-in-every-flit
title: "ADR-0008 VCID in Every Flit"
aliases: ["Decision 0008","Four-bit VCID decision"]
type: decision
status: accepted
layers: ["L2"]
tags: ["gnet","gnet/decision","gnet/status/accepted","gnet/layer/l2"]
parent: "[[Decisions MOC]]"
related: ["[[Virtual Channels and VCIDs]]","[[32-bit Flit Format]]","[[Direct Link Protocol]]"]
updated: 2026-09-02
---
# Decision 0008: Reserve four bits of every flit for VCID

> [!info] Knowledge graph
> **Up:** [[Decisions MOC]] · **Related:** [[Virtual Channels and VCIDs]] · [[32-bit Flit Format]] · [[Direct Link Protocol]]

Status: **ACCEPTED**

## Decision

Every transmitted GNet flit is exactly 32 bits. The most-significant four bits are a hop-local Virtual Channel Identifier and the remaining 28 bits carry DLP header, protocol payload, padding, or trailer data.

```text
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |                  Carried bits [27:0]                  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

The VCID is part of the 32-bit flit, not sideband. Protocol bitstreams are packed continuously into successive 28-bit carried regions and may cross octet and logical-word boundaries.

VCIDs are local to a link, direction, and any physical sender/channel context supplied by a shared medium. They identify active bounded DLP segments, may be reused after completion, and are replaced at each router. Multiple active VCIDs allow a link scheduler to interleave latency-sensitive and bulk traffic.

## Consequences

- At most 16 VCID values exist in the base format.
- No transmitted flit contains 32 protocol-payload bits.
- Packet documents must distinguish logical protocol layout from actual 4+28 DLP flit packing.
- Hardware requires per-VCID receive, length, integrity, and timeout state.
- Link efficiency before headers and trailers is 87.5 percent.
- Bounded segments prevent one transfer from owning a VCID or wormhole path indefinitely.

The allocation of individual VCID values, exact segment limit, explicit versus implicit allocation, persistent Flow ID binding, and error recovery remain separate open decisions.
