---
id: gnet-a
title: "GNET-A"
aliases: ["GNet residential access"]
type: media
status: draft
layers: ["L1","L2"]
tags: ["gnet","gnet/media","gnet/status/draft","gnet/layer/l1","gnet/layer/l2"]
parent: "[[Media and Links MOC]]"
related: ["[[Direct Link Protocol]]","[[Deployment Topology]]"]
updated: 2026-09-02
---
# GNET-A residential access

> [!info] Knowledge graph
> **Up:** [[Media and Links MOC]] · **Related:** [[Direct Link Protocol]] · [[Deployment Topology]]


Status: **ACCEPTED concept; OPEN detailed specification**

GNET-A is a centrally polled shared access system for approximately 256 homes or premises. Its target aggregate rate is 10 Mb/s. The access controller owns upstream transmission opportunities, making collisions impossible and allowing capacity to be reserved for voice or other real-time GDP flows.

GNET-A carries ordinary DLP frames. It must define premise/channel identity, polling cycle, ranging/timing, downstream selection, admission and scheduling of reserved capacity, failure isolation, and privacy between premises.

The physical plant, modulation, coding, reach, repeater plan, and exact sharing hierarchy remain OPEN. GNET-A must not introduce a second network-layer address architecture.
