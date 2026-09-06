---
id: adr-0011-baseline-vc2-flit-without-sof
title: "ADR-0011 Baseline VC2 Flit Without SOF"
aliases: ["Decision 0011","VC2 baseline"]
type: decision
status: accepted
layers: ["L1","L2"]
tags: ["gnet","gnet/decision","gnet/status/accepted","gnet/layer/l2"]
parent: "[[Decisions MOC]]"
related: ["[[34-bit Flit Format]]","[[Virtual Channels and VCIDs]]","[[ADR-0017 34-bit Flit Format]]","[[ADR-0008 VCID in Every Flit]]"]
updated: 2026-09-06
---
# Decision 0011: Baseline VC2 flit has no SOF bit

Status: **ACCEPTED 2026-09-03**

## Decision

The baseline physical flit is:

```text
34 bits total
2-bit VCID
32 carried bits
no SOF bit
```

The first data flit received on an inactive allocated VC implicitly begins a DLP segment. Completion, ABORT, timeout, or reset releases that VC context.

GNet-3 and GNet-10 use this VC2 profile. The current GNet-20 concept also assumes VC2.

A future advanced profile MAY negotiate:

```text
VC4 = 4-bit VCID + 28 carried bits
```

but no endpoint may assume VC4 before successful capability negotiation.

## Consequences

- four baseline wire VCIDs are available;
- baseline carried capacity is 32 bits/flit;
- start state is derived from link-control allocation plus VC activity instead of spending a permanent SOF bit;
- packet/protocol fields may cross physical-flit boundaries.

This supersedes the field-allocation parts of ADR-0008. ADR-0017 supersedes the former 32-bit total-width decision.
