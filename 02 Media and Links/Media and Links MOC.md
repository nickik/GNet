---
id: media-and-links-moc
title: "Media and Links MOC"
aliases: ["Media Map of Content"]
type: moc
status: active
tags: ["gnet","gnet/moc","gnet/media","gnet/status/active"]
parent: "[[GNet Home]]"
related: ["[[Architecture MOC]]","[[Packet Formats MOC]]","[[Implementation MOC]]"]
updated: 2026-09-03
---
# Media and links map of content

## Native local copper

- [[GNet PHY Profiles]] — GNet-3, GNet-10, and future GNet-20.
- [[GNet Copper Cabling]] — installation grades and certified-cable direction.
- [[GNet Modular Connector]] — GMC-8 mechanical/electrical connector family.
- [[GNet Link Control Protocol]] — bootstrap, credits, grants, VC control, reset/status.
- [[GNet Coupler]] — GC3 shared-medium behavior.
- [[GNet Switch]] — GS3/GS10 switched behavior.
- [[GNET-L]] — local-copper family landing page.

## Common data path

- [[32-bit Flit Format]] — baseline VC2 + 30 carried bits.
- [[Virtual Channels and VCIDs]] — hop-local VC allocation and lifecycle.
- [[Direct Link Protocol]] — minimal L2 data-path contract.
- [[DLP Segment Size Classes]] — superseded historical DLP size-class model.

## Other media

- [[GNET-A]] — centrally scheduled residential access.
- [[GNET-P]] — point-to-point infrastructure/trunk family.

Key decisions: [[ADR-0007 32-bit Flit Format]], [[ADR-0011 Baseline VC2 Flit Without SOF]], [[ADR-0012 Minimum GNet-3 Compatibility Profile]], [[ADR-0013 Receiver Credits and Infrastructure Grants]], and [[ADR-0014 GC3 GS3 GS10 LAN Ladder]].
