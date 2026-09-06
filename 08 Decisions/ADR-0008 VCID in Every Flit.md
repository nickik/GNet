---
id: adr-0008-vcid-in-every-flit
title: "ADR-0008 VCID in Every Flit"
aliases: ["Decision 0008","VCID decision"]
type: decision
status: superseded
layers: ["L2"]
tags: ["gnet","gnet/decision","gnet/status/superseded","gnet/layer/l2"]
parent: "[[Decisions MOC]]"
related: ["[[Virtual Channels and VCIDs]]","[[32-bit Flit Format]]","[[ADR-0011 Baseline VC2 Flit Without SOF]]","[[ADR-0017 32-bit Data Flit and PHY Phits]]"]
updated: 2026-09-06
---
# Decision 0008: VCID in every flit — historical evolution

Status: **SUPERSEDED by ADR-0011 and ADR-0017**

This decision established the important architectural requirement that every hop-local data flit has an unambiguous VC identity. That requirement remains, but the historical choice to place VCID bits *inside* a fixed-width transmitted flit does not.

Its packed field allocation changed during design:

1. original form: `VCID:4 + carried:28`;
2. intermediate amendment: `VCID:2 + SOF:1 + carried:29`;
3. ADR-0011 form: `VCID:2 + carried:30`, with no SOF bit.

[[ADR-0017 32-bit Data Flit and PHY Phits]] replaces all of those packed layouts with a 32-bit data flit plus separately associated hop-local VC metadata. Baseline VC2 remains two bits; the PHY defines how that metadata is conveyed with the flit.

This file preserves the design history rather than silently rewriting it.
