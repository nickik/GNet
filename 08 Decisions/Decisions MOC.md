---
id: decisions-moc
title: "Decisions MOC"
aliases: ["ADR Index","Decision Map of Content"]
type: moc
status: active
tags: ["gnet","gnet/moc","gnet/decision","gnet/status/active"]
parent: "[[GNet Home]]"
related: ["[[Architecture MOC]]","[[Open Questions]]"]
updated: 2026-09-06
---
# Decisions map of content

## Current backbone

- [[ADR-0001 Layer Boundaries]]
- [[ADR-0002 Minimal GDP Header]]
- [[ADR-0003 Bounded Discovery]]
- [[ADR-0004 Direct Point-to-Point Link]]
- [[ADR-0005 Tunnels and Streams]]
- [[ADR-0006 GNET-L Rate]]
- [[ADR-0012 Minimum GNet-3 Compatibility Profile]]
- [[ADR-0013 Receiver Credits and Infrastructure Grants]]
- [[ADR-0014 GC3 GS3 GS10 LAN Ladder]]
- [[ADR-0015 Restore Minimal GDP Header]]
- [[ADR-0016 Access and Carrier Profile Naming]]
- [[ADR-0017 32-bit Data Flit and PHY Phits]]

## Superseded/history retained

- [[ADR-0007 32-bit Flit Format]] — historical 32-bit **total link-transfer** width; superseded by ADR-0017's 32-bit **data flit** plus PHY phit model.
- [[ADR-0008 VCID in Every Flit]] — historical packed VCID layouts; VC association survives but packing is superseded by ADR-0017.
- [[ADR-0009 No GDP Integrity Field]] — historical path; no-checksum outcome restored by ADR-0015.
- [[ADR-0010 DLP Segment Size Classes]] — DLP size classes replaced by GDP Size Class.
- [[ADR-0011 Baseline VC2 Flit Without SOF]] — historical `2-bit VCID + 30 carried bits` allocation superseded by ADR-0017; no-SOF and implicit segment-start behavior survive.

Create a superseding ADR rather than silently rewriting an accepted architectural choice.
