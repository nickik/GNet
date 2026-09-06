---
id: virtual-channels-and-vcids
title: "Virtual Channels and VCIDs"
aliases: ["VCID","GNet virtual channels","Virtual Channel Identifier"]
type: protocol
status: accepted
layers: ["L2"]
tags: ["gnet","gnet/protocol","gnet/status/accepted","gnet/layer/l2"]
parent: "[[Media and Links MOC]]"
related: ["[[Direct Link Protocol]]","[[32-bit Flit Format]]","[[GNet Link Control Protocol]]","[[ADR-0017 32-bit Data Flit and PHY Phits]]"]
updated: 2026-09-06
---
# Virtual channels and VCIDs

> [!info] Knowledge graph
> **Up:** [[Media and Links MOC]] · **Related:** [[Direct Link Protocol]] · [[32-bit Flit Format]] · [[GNet Link Control Protocol]]

Status: **ACCEPTED baseline; wider future VC options not defined**

The baseline Virtual Channel Identifier (VCID) is **two bits of hop-local link metadata associated with every 32-bit data flit**. It gives four hop-local wire VCIDs per link.

```text
VC metadata: [ VCID:2 ]
Flit data:   [ Data:32 ]
```

The VCID does not reduce the 32-bit flit data width. A PHY may carry the VCID inline, in sideband signaling, or by another profile-defined mapping, provided the VC identity of every transferred flit is unambiguous.

There is **no SOF bit**. Once a VC has been allocated for a transfer, the first data flit received while that VC is inactive implicitly begins the DLP segment. Completion, ABORT, timeout, or link reset releases its active state.

## Meaning

A VCID is:

- local to one link and direction;
- associated with every flit belonging to one active hop-local transfer;
- allocated/released by the link-control mechanism;
- available to let infrastructure pause one transfer and service another without confusing receiver state;
- replaced or terminated at a forwarding node;
- unrelated to GDP addresses, tunnels, application sessions, or ports.

The baseline profile intentionally provides only four wire VCIDs. Minimum GNet-3 requires enough implementation context for at least one paused NORMAL receive transfer and one REALTIME receive transfer; it does not require four simultaneously buffered full packets.

## Wider future VC identifiers

A future link profile MAY define a negotiated wider VC identifier, for example VC4 or VC8. Such a profile is a future option only:

- no current baseline endpoint may assume VC4 or VC8;
- GNet-3 and GNet-10 use VC2;
- widening the VCID MUST NOT change the 32-bit flit data width;
- the future PHY/profile must define how the wider VC metadata is carried and negotiated.

See [[ADR-0017 32-bit Data Flit and PHY Phits]].
