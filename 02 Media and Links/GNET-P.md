---
id: gnet-p
title: "GNET-P"
aliases: ["GNet point-to-point trunk","GNet infrastructure link"]
type: media
status: draft
layers: ["L1","L2"]
tags: ["gnet","gnet/media","gnet/status/draft","gnet/layer/l1","gnet/layer/l2"]
parent: "[[Media and Links MOC]]"
related: ["[[Direct Link Protocol]]","[[Virtual Channels and VCIDs]]","[[Deployment Topology]]"]
updated: 2026-09-03
---
# GNET-P point-to-point trunk

Status: **ACCEPTED separate infrastructure concept and rate family; OPEN detailed framing/PHY**

GNET-P provides dedicated synchronous point-to-point infrastructure links at 10, 25, and 50 Mbit/s. Initial systems may use coaxial cable; later systems may use fiber without changing GDP.

GNET-P is **not GNet-10 LAN**. The names currently overlap numerically:

- **GNet-10** — 10 Mbit/s switched four-pair LAN profile on GS10;
- **GNet Link/10 CX / GNET-P 10** — separate point-to-point infrastructure trunk profile.

The trunk naming should be revisited before commercial release to avoid customer ambiguity; the distinct technical concept is retained for now.

Each direction has an independent hop-local VC namespace. The baseline logical flit is VC2 (`2-bit VCID + 30 carried bits`) unless a future advanced width is explicitly negotiated. GNET-P must preserve the same receiver-credit invariant as other GNet hops: one credit represents one physical flit of guaranteed downstream capacity.

Clock recovery, line code, keepalive, protection switching, control carriage, exact link integrity, and coax-to-fiber transition rules remain OPEN.
