---
id: virtual-channels-and-vcids
title: "Virtual Channels and VCIDs"
aliases: ["VCID","GNet virtual channels","Virtual Channel Identifier"]
type: protocol
status: accepted
layers: ["L2"]
tags: ["gnet","gnet/protocol","gnet/status/accepted","gnet/layer/l2"]
parent: "[[Media and Links MOC]]"
related: ["[[Direct Link Protocol]]","[[32-bit Flit Format]]","[[GNet Link Control Protocol]]","[[ADR-0011 Baseline VC2 Flit Without SOF]]"]
updated: 2026-09-03
---
# Virtual channels and VCIDs

> [!info] Knowledge graph
> **Up:** [[Media and Links MOC]] · **Related:** [[Direct Link Protocol]] · [[32-bit Flit Format]] · [[GNet Link Control Protocol]]

Status: **ACCEPTED baseline; advanced widths DRAFT**

The baseline Virtual Channel Identifier (VCID) is the **two-bit field at the start of every 32-bit flit**. It gives four hop-local wire VCIDs per link.

```text
[ VCID:2 | Carried bits:30 ]
```

There is **no SOF bit**. Once a VC has been allocated for a transfer, the first data flit received while that VC is inactive implicitly begins the DLP segment. Completion, ABORT, timeout, or link reset releases its active state.

## Meaning

A VCID is:

- local to one link and direction;
- repeated on the flits belonging to one active hop-local transfer;
- allocated/released by the link-control mechanism;
- available to let infrastructure pause one transfer and service another without confusing their receiver state;
- replaced or terminated at a forwarding node;
- unrelated to GDP addresses, tunnels, application sessions, or ports.

The baseline profile intentionally provides only four wire VCIDs. Minimum GNet-3 requires enough implementation context for at least one paused NORMAL receive transfer and one REALTIME receive transfer; it does not require four simultaneously buffered full packets.

## Wider future VCID

Advanced future links MAY negotiate a wider wire VCID after baseline link establishment:

```text
VC2 = 2-bit VCID + 30 carried bits
VC4 = 4-bit VCID + 28 carried bits
```

GNet-3 and GNet-10 use VC2. The current GNet-20 concept also assumes VC2. VC4 is reserved for later advanced switched or cluster profiles and is not an independent minimum compatibility mode.

See [[ADR-0011 Baseline VC2 Flit Without SOF]].
