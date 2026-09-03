---
id: adr-0008-vcid-in-every-flit
title: "ADR-0008 VCID in Every Flit"
aliases: ["Decision 0008","VCID decision"]
type: decision
status: accepted
layers: ["L2"]
tags: ["gnet","gnet/decision","gnet/status/accepted","gnet/layer/l2"]
parent: "[[Decisions MOC]]"
related: ["[[Virtual Channels and VCIDs]]","[[32-bit Flit Format]]","[[Direct Link Protocol]]"]
updated: 2026-09-03
---
# Decision 0008: Reserve two bits of every flit for VCID

Status: **ACCEPTED; amended 2026-09-03**

Every transmitted 32-bit flit begins with a **2-bit hop-local VCID**, followed by a one-bit SOF marker and 29 carried bits:

```text
[ VCID:2 | SOF:1 | Carried bits:29 ]
```

The two-bit VCID provides four virtual channels on each direct link. VCIDs are local to one link/direction and are terminated or reassigned by forwarding nodes. They permit flit interleaving and preemption between independent traffic classes/flows without consuming a separate sideband field.

The earlier four-bit VCID allocation is superseded. The reduction to two bits recovers two carried bits per flit and is sufficient for the current link scheduling model.
