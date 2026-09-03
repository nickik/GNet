---
id: adr-0008-vcid-in-every-flit
title: "ADR-0008 VCID in Every Flit"
aliases: ["Decision 0008","VCID decision"]
type: decision
status: superseded
layers: ["L2"]
tags: ["gnet","gnet/decision","gnet/status/superseded","gnet/layer/l2"]
parent: "[[Decisions MOC]]"
related: ["[[Virtual Channels and VCIDs]]","[[32-bit Flit Format]]","[[ADR-0011 Baseline VC2 Flit Without SOF]]"]
updated: 2026-09-03
---
# Decision 0008: VCID in every flit — historical evolution

Status: **SUPERSEDED by ADR-0011**

This decision established the important architectural principle that VCID is part of every physical flit and is hop/link-local. That principle remains.

Its field allocation changed during design:

1. original form: `VCID:4 + carried:28`;
2. intermediate amendment: `VCID:2 + SOF:1 + carried:29`;
3. current baseline: `VCID:2 + carried:30`, with **no SOF bit**.

The current normative layout and implicit first-flit rule are recorded in [[ADR-0011 Baseline VC2 Flit Without SOF]]. This file preserves the decision history rather than silently rewriting it.
