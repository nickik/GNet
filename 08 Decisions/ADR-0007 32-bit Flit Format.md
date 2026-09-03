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
updated: 2026-09-02
---
# Decision 0007: Show and package the protocol as 32-bit flits

> [!info] Knowledge graph
> **Up:** [[Decisions MOC]] · **Related:** [[32-bit Flit Format]] · [[Direct Link Protocol]] · [[ADR-0008 VCID in Every Flit]]


Status: **ACCEPTED presentation unit; DRAFT DLP trailer details**

All GNet wire-format documents use RFC-style bit diagrams. Where a row is identified as a transmitted flit, it is exactly 32 bits and includes the four-bit VCID defined by [[ADR-0008 VCID in Every Flit]]. Logical protocol layouts must be labelled as logical words rather than flits. Packet definitions must show or reference how the logical bitstream crosses 28-bit carried-region boundaries.

Multi-octet fields use network byte order. DLP supplies the exact payload-octet length, the last 28-bit carried region is zero-padded, and a final flit carries the DLP integrity trailer. The integrity algorithm and trailer allocation remain open.

A 32-bit flit is the complete logical link-transfer unit, not 32 payload bits plus sideband. This decision does not require 32 parallel physical wires; each medium may serialize and encode the flit differently.
