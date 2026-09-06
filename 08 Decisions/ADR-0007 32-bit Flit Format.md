---
id: adr-0007-32-bit-flit-format
title: "ADR-0007 32-bit Flit Format"
aliases: ["Decision 0007"]
type: decision
status: superseded
layers: ["L1","L2"]
tags: ["gnet","gnet/decision","gnet/status/superseded","gnet/layer/l1","gnet/layer/l2"]
parent: "[[Decisions MOC]]"
related: ["[[32-bit Flit Format]]","[[ADR-0011 Baseline VC2 Flit Without SOF]]","[[ADR-0017 32-bit Data Flit and PHY Phits]]"]
updated: 2026-09-06
---
# Decision 0007: Historical 32-bit total link-transfer width

Status: **SUPERSEDED by ADR-0017**

This decision originally froze every transmitted GNet flit as exactly 32 logical link bits. Physical media could serialize or line-code those bits differently, but the complete link-transfer unit was constrained to 32 bits.

The field split evolved historically through several layouts, ending with the ADR-0011 baseline:

```text
[ VCID:2 | carried:30 ]
```

[[ADR-0017 32-bit Data Flit and PHY Phits]] supersedes the **32-bit total-width** interpretation. The current architecture instead defines:

- a GNet **flit** as exactly 32 data bits;
- hop-local VC metadata associated with each flit outside those 32 data bits;
- a PHY-specific **phit** as the physical transfer unit.

The useful architectural intent of a natural 32-bit data unit is retained, but 32 bits is no longer the total amount of link information associated with a baseline VC2 flit.
