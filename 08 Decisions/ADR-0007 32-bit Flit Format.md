---
id: adr-0007-32-bit-flit-format
title: "ADR-0007 32-bit Flit Format"
aliases: ["Decision 0007"]
type: decision
status: accepted
layers: ["L2"]
tags: ["gnet","gnet/decision","gnet/status/accepted","gnet/layer/l2"]
parent: "[[Decisions MOC]]"
related: ["[[32-bit Flit Format]]","[[ADR-0008 VCID in Every Flit]]","[[ADR-0011 Baseline VC2 Flit Without SOF]]"]
updated: 2026-09-03
---
# Decision 0007: Use a 32-bit flit as the complete link-transfer unit

Status: **ACCEPTED; width remains normative**

Every transmitted GNet flit is exactly 32 logical bits. Physical media may serialize or line-code those bits differently.

This ADR freezes the **32-bit total width**, not the internal metadata split. Earlier revisions described 4+28 and later 2+1+29 layouts. The current baseline allocation is defined by [[ADR-0011 Baseline VC2 Flit Without SOF]]:

```text
[ VCID:2 | carried:30 ]
```

Historical layout changes do not supersede the 32-bit-width decision itself.
