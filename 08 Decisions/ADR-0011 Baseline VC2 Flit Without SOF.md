---
id: adr-0011-baseline-vc2-flit-without-sof
title: "ADR-0011 Baseline VC2 Flit Without SOF"
aliases: ["Decision 0011","VC2 baseline"]
type: decision
status: superseded
layers: ["L1","L2"]
tags: ["gnet","gnet/decision","gnet/status/superseded","gnet/layer/l2"]
parent: "[[Decisions MOC]]"
related: ["[[32-bit Flit Format]]","[[Virtual Channels and VCIDs]]","[[ADR-0007 32-bit Flit Format]]","[[ADR-0008 VCID in Every Flit]]","[[ADR-0017 32-bit Data Flit and PHY Phits]]"]
updated: 2026-09-06
---
# Decision 0011: Historical packed VC2 flit without SOF

Status: **SUPERSEDED by ADR-0017**

## Historical decision

ADR-0011 defined the baseline transmitted unit as:

```text
32 bits total
2-bit VCID
30 carried bits
no SOF bit
```

The first data flit received on an inactive allocated VC implicitly began a DLP segment. Completion, ABORT, timeout, or reset released that VC context.

## What survives

[[ADR-0017 32-bit Data Flit and PHY Phits]] supersedes the `2 + 30 = 32 total` field allocation but retains:

- baseline VC2 with four hop-local wire VCIDs;
- a VC identity associated with every data flit;
- no SOF bit;
- implicit segment start on the first flit of an inactive allocated VC;
- VC termination/reassignment at forwarding nodes.

The current baseline is therefore **32 data bits per flit plus an associated 2-bit VCID**, with the PHY defining how the two are physically conveyed.

VC4 or VC8 may exist only as future negotiated wider-VC options and must retain the 32-bit flit data width.

## Consequence of supersession

The old requirement that protocol fields be packed across 30-bit carried regions is removed. Current 32-bit protocol words can align naturally with 32-bit data flits.
