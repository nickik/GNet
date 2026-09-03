---
id: adr-0010-dlp-segment-size-classes
title: "ADR-0010 DLP Segment Size Classes"
aliases: ["Decision 0010", "Old DLP package size classes"]
type: decision
status: superseded
layers: ["L2","L3"]
tags: ["gnet","gnet/decision","gnet/status/superseded","gnet/layer/l2","gnet/layer/l3"]
parent: "[[Decisions MOC]]"
related: ["[[Direct Link Protocol]]","[[GDP Datagram]]"]
updated: 2026-09-03
---
# Decision 0010: Bounded DLP segment size classes

Status: **SUPERSEDED 2026-09-03**

The earlier design put a two-bit size class and explicit payload length into DLP. That model is no longer normative.

The current design keeps DLP minimal and moves fixed packet sizing to GDP. GDP carries a **4-bit Size Class** in its first header flit, giving sixteen payload classes from empty/tiny control packets through 1 MiB jumbograms.

See [[GDP Datagram]] for the current size-class registry and [[Direct Link Protocol]] for the simplified Layer-2 model.
