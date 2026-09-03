---
id: gnet-l
title: "GNET-L"
aliases: ["GNet local star"]
type: media
status: accepted
layers: ["L1","L2"]
tags: ["gnet","gnet/media","gnet/status/accepted","gnet/layer/l1","gnet/layer/l2"]
parent: "[[Media and Links MOC]]"
related: ["[[Direct Link Protocol]]","[[Virtual Channels and VCIDs]]","[[ADR-0006 GNET-L Rate]]"]
updated: 2026-09-02
---
# GNET-L local star

> [!info] Knowledge graph
> **Up:** [[Media and Links MOC]] · **Related:** [[Direct Link Protocol]] · [[Virtual Channels and VCIDs]] · [[ADR-0006 GNET-L Rate]]


Status: **ACCEPTED physical plan; OPEN electrical encoding**

GNET-L connects terminals, workstations, minicomputers, and local servers to a passive or active star hub. The current target line rate is **3 Mb/s**.

## Four-pair connector plan

| Pair | Direction/function |
|---|---|
| 1 | endpoint request / hub permission control |
| 2 | endpoint-to-hub data |
| 3 | hub-to-endpoint data |
| 4 | reserved for later clocking, power/control, redundancy, or increased capacity |

The connector family is the eight-position modular connector commonly described as RJ-45; the exact pinout is OPEN.

An endpoint raises REQUEST. The hub selects a requester and returns PERMISSION; only the granted endpoint transmits upstream. Downstream delivery is dedicated per star leg. This is collision-free and gives the hub intrinsic physical port identity.

Every transferred 32-bit flit contains a 4-bit VCID and 28 carried bits. The physical star leg supplies endpoint identity; the VCID identifies one active bounded DLP segment in that direction and permits the scheduler to interleave control, real-time, and bulk traffic.

Passive hubs are planned in 4/8/16/32/64-port forms. An active hub/router adds buffering, GDP forwarding, at least one uplink, and a management console. Electrical loading and whether larger passive sizes require repeated stages are OPEN.
