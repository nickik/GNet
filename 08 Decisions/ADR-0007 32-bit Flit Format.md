---
id: adr-0007-32-bit-flit-format
title: "ADR-0007 32-bit Flit Format"
aliases: ["Decision 0007"]
type: decision
status: accepted
layers: ["L2"]
tags: ["gnet","gnet/decision","gnet/status/accepted","gnet/layer/l2"]
parent: "[[Decisions MOC]]"
related: ["[[32-bit Flit Format]]","[[Direct Link Protocol]]"]
updated: 2026-09-02
---
# Decision 0007: Show and package the protocol as 32-bit flits

> [!info] Knowledge graph
> **Up:** [[Decisions MOC]] · **Related:** [[32-bit Flit Format]] · [[Direct Link Protocol]]


Status: **ACCEPTED presentation unit; DRAFT DLP trailer details**

All GNet wire-format documents use RFC-style bit diagrams in which one row is exactly one 32-bit flit. Packet definitions must show how every field crosses flit boundaries rather than replacing the layout with offset/size Markdown tables.

Multi-octet fields and multi-flit integers use network byte order. DLP supplies the exact payload-octet length, the last payload flit is zero-padded, and a final flit carries CRC-8 plus reserved zero bits. The CRC polynomial and whether the final trailer can later be compressed by a medium-specific profile remain open.

A 32-bit flit is a logical transmission unit. This decision does not require 32 parallel physical wires; each medium may serialize and encode the flit differently.
