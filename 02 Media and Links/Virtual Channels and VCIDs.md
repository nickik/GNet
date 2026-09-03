---
id: virtual-channels-and-vcids
title: "Virtual Channels and VCIDs"
aliases: ["VCID","GNet virtual channels","Virtual Channel Identifier"]
type: protocol
status: draft
layers: ["L2"]
tags: ["gnet","gnet/protocol","gnet/status/draft","gnet/layer/l2"]
parent: "[[Media and Links MOC]]"
related: ["[[Direct Link Protocol]]","[[32-bit Flit Format]]","[[Transport and Flows]]","[[ADR-0008 VCID in Every Flit]]"]
updated: 2026-09-03
---
# Virtual channels and VCIDs

> [!info] Knowledge graph
> **Up:** [[Media and Links MOC]] · **Related:** [[Direct Link Protocol]] · [[32-bit Flit Format]] · [[ADR-0008 VCID in Every Flit]]

Status: **CURRENT DRAFT**

The Virtual Channel Identifier (VCID) is the **two-bit field at the start of every 32-bit flit**. It gives four hop-local virtual channels per link.

```text
[ VCID:2 | SOF:1 | Carried bits:29 ]
```

## Meaning

A VCID is:

- local to one link and one direction;
- repeated on the flits belonging to that active transfer;
- available to let the scheduler interleave or preempt traffic on the same physical link;
- replaced or terminated at a router;
- unrelated to GDP addresses, GNet Tunnel IDs, application sessions, or ports.

The VCID is intentionally small. Four values are sufficient for the current scheduling model while preserving 29 carried bits in every flit.

## Relationship to SOF and Frame Type

SOF identifies the first flit of a frame. On an SOF flit, the first carried bit is the one-bit Frame Type discriminator. Continuation flits retain all 29 carried bits for protocol data.

## Open control questions

The allocation policy for the four VCID values, timeout/recovery behavior, and exact preemption/credit rules remain profile-specific design work. The base protocol only fixes the two-bit field and its hop-local meaning.
