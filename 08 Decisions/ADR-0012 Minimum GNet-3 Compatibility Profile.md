---
id: adr-0012-minimum-gnet-3-compatibility-profile
title: "ADR-0012 Minimum GNet-3 Compatibility Profile"
aliases: ["Decision 0012","Minimum GNet-3"]
type: decision
status: accepted
layers: ["L1","L2"]
tags: ["gnet","gnet/decision","gnet/status/accepted","gnet/nic"]
parent: "[[Decisions MOC]]"
related: ["[[Minimum GNet-3 NIC]]","[[GNet PHY Profiles]]","[[GNet Link Control Protocol]]"]
updated: 2026-09-03
---
# Decision 0012: Minimum GNet-3 is the universal native compatibility profile

Status: **ACCEPTED 2026-09-03**

Every native GNet NIC begins in Minimum GNet-3 compatibility, including later GNet-10, GNet-20, server, router, cluster, and high-performance adapters.

Minimum GNet-3 includes:

- four-pair GMC-8 copper attachment;
- 3 Mbit/s nominal data mode plus 1.5 and 0.75 Mbit/s fallback;
- VC2 flits (`2 + 30`, no SOF);
- GLCP control;
- per-flit receiver credits;
- NORMAL and REALTIME priorities;
- at least two concurrent active receive contexts.

Advanced capability is negotiated after baseline establishment. No incompatible "Minimum GNet-10" profile exists.

This gives every generation a simple safe common mode and permits incremental infrastructure upgrades without replacing all endpoints simultaneously.
