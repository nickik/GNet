---
id: media-and-links-moc
title: "Media and Links MOC"
aliases: ["Media Map of Content"]
type: moc
status: active
tags: ["gnet", "gnet/moc", "gnet/media", "gnet/status/active"]
parent: "[[GNet Home]]"
related: ["[[Architecture MOC]]", "[[Packet Formats MOC]]"]
updated: 2026-09-02
---
# Media and links map of content

> [!info] Knowledge graph
> **Up:** [[GNet Home]] · **Related:** [[Architecture MOC]] · [[Packet Formats MOC]]

- [[Direct Link Protocol]] — common L2 bounded-segment envelope and link integrity.
- [[Virtual Channels and VCIDs]] — 4-bit hop-local multiplexing and per-VCID state.
- [[DLP Segment Size Classes]] — bounded 64-, 256-, and 1,024-octet DLP payload classes.
- [[GNET-L]] — local four-pair request/grant star.
- [[GNET-A]] — centrally scheduled residential access.
- [[GNET-P]] — synchronous routed trunks.
- [[32-bit Flit Format]] — exact 4-bit VCID plus 28-bit carried-data format.

Key decisions: [[ADR-0004 Direct Point-to-Point Link]], [[ADR-0006 GNET-L Rate]], [[ADR-0007 32-bit Flit Format]], [[ADR-0008 VCID in Every Flit]], and [[ADR-0010 DLP Segment Size Classes]].
