---
id: adr-0007-32-bit-flit-format
title: "ADR-0007 32-bit Flit Format"
aliases: ["Decision 0007"]
type: decision
status: accepted
layers: ["L2"]
tags: ["gnet","gnet/decision","gnet/status/accepted","gnet/layer/l2"]
parent: "[[Decisions MOC]]"
related: ["[[32-bit Flit Format]]","[[Direct Link Protocol]]","[[ADR-0008 VCID in Every Flit]]"]
updated: 2026-09-03
---
# Decision 0007: Use a 32-bit flit as the complete link-transfer unit

Status: **ACCEPTED; amended 2026-09-03**

Every transmitted flit is exactly 32 bits. The current base layout is:

```text
[ VCID:2 | SOF:1 | Carried bits:29 ]
```

VCID and SOF are part of the 32-bit flit and are not sideband. On the first flit only, the first carried bit is the one-bit Frame Type discriminator, leaving 28 protocol-specific bits in that first flit. Continuation flits carry 29 protocol bits.

The physical medium may serialize or encode the logical flit differently. Packet documents must distinguish logical protocol fields from transmitted flits and show the actual flit mapping where it matters.

This amendment supersedes the earlier 4-bit-VCID/28-carried-bit allocation while retaining the 32-bit total flit width.
