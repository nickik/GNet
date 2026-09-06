---
id: adr-0017-34-bit-flit-format
title: "ADR-0017 34-bit Flit Format"
aliases: ["Decision 0017"]
type: decision
status: accepted
layers: ["L2"]
tags: ["gnet","gnet/decision","gnet/status/accepted","gnet/layer/l2"]
parent: "[[Decisions MOC]]"
related: ["[[34-bit Flit Format]]","[[ADR-0008 VCID in Every Flit]]","[[ADR-0011 Baseline VC2 Flit Without SOF]]"]
updated: 2026-09-06
---
# Decision 0017: Use a 34-bit flit as the complete link-transfer unit

Status: **ACCEPTED 2026-09-06 — supersedes ADR-0007**

Every transmitted baseline GNet flit is exactly 34 logical bits. Physical media may serialize or line-code those bits differently.

The baseline allocation is defined by [[ADR-0011 Baseline VC2 Flit Without SOF]]:

```text
[ VCID:2 | carried:32 ]
```

The two-bit hop-local VCID is metadata in every flit; it is not part of the 32 carried bits. The 32-bit carried region intentionally aligns the GDP logical words and removes artificial bit-crossing from the baseline profile.

This decision supersedes the former 32-bit total-width decision.
