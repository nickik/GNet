---
id: adr-0012-minimum-gnet-3-compatibility-profile
title: "ADR-0012 Minimum GNet-3 Compatibility Profile"
aliases: ["Decision 0012","Minimum GNet-3"]
type: decision
status: accepted
layers: ["L1","L2"]
tags: ["gnet","gnet/decision","gnet/status/accepted","gnet/nic"]
parent: "[[Decisions MOC]]"
related: ["[[Minimum GNet-3 NIC]]","[[GNet PHY Profiles]]","[[GNet Link Control Protocol]]","[[ADR-0017 32-bit Data Flit and PHY Phits]]"]
updated: 2026-09-06
---
# Decision 0012: Minimum GNet-3 is the universal native compatibility profile

Status: **ACCEPTED 2026-09-03; flit wording updated by ADR-0017**

Every native GNet NIC begins in Minimum GNet-3 compatibility, including later GNet-10, GNet-20, server, router, cluster, and high-performance adapters.

Minimum GNet-3 includes:

- four-pair GMC-8 copper attachment;
- 3 Mbit/s nominal flit-data mode plus 1.5 and 0.75 Mbit/s fallback;
- exactly 32 data bits per GNet flit;
- baseline VC2 metadata associated with every flit, providing four wire VCs;
- no SOF bit;
- GLCP control;
- per-flit receiver credits;
- NORMAL and REALTIME priorities;
- at least two concurrent active receive contexts.

The physical mapping of a flit and its VC2 metadata into phits is defined by the GNet-3 PHY profile.

Advanced capability is negotiated after baseline establishment. No incompatible "Minimum GNet-10" profile exists. Future VC4 or VC8 modes, if standardized, are advanced options only and must retain the 32-bit flit data width.

This gives every generation a simple safe common mode and permits incremental infrastructure upgrades without replacing all endpoints simultaneously.
