---
id: gnet-a
title: "GNET-A"
aliases: ["GNet residential access"]
type: media
status: draft
layers: ["L1","L2"]
tags: ["gnet","gnet/media","gnet/status/draft","gnet/layer/l1","gnet/layer/l2"]
parent: "[[Media and Links MOC]]"
related: ["[[Direct Link Protocol]]","[[Virtual Channels and VCIDs]]","[[Deployment Topology]]"]
updated: 2026-09-03
---
# GNET-A residential access

Status: **ACCEPTED concept; OPEN detailed specification**

GNET-A is a centrally scheduled shared access system for approximately 256 homes or premises. Its target aggregate rate is 10 Mbit/s. The access controller owns upstream transmission opportunities, avoiding collision-based access and allowing reserved real-time capacity.

GNET-A carries ordinary GNet data using the current baseline logical flit unless its profile explicitly negotiates something else:

```text
[ VCID:2 | Carried bits:30 ]
```

A polling/grant channel supplies premise identity; the VCID remains hop-local transfer state, not a customer/global address.

The physical plant, modulation, coding, ranging, privacy, scheduling, reach, repeater plan, and exact control mapping remain OPEN. GNET-A must not introduce a second GDP addressing architecture.
