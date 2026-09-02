---
id: adr-0004-direct-point-to-point-link
title: "ADR-0004 Direct Point-to-Point Link"
aliases: ["Decision 0004"]
type: decision
status: frozen
layers: ["L2"]
tags: ["gnet","gnet/decision","gnet/status/frozen","gnet/layer/l2"]
parent: "[[Decisions MOC]]"
related: ["[[Direct Link Protocol]]","[[GNET-L]]"]
updated: 2026-09-02
---
# Decision 0004: Direct links have no network identity

> [!info] Knowledge graph
> **Up:** [[Decisions MOC]] · **Related:** [[Direct Link Protocol]] · [[GNET-L]]


Status: **FROZEN**

DLP assumes one directly attached endpoint per logical line. It provides byte/frame delimiting and CRC-8 but no source/destination address and no session state. Physical port or channel identity determines local delivery.

An earlier name, GPP (GNet Point-to-Point), described the same design direction. The accepted layer names are DLP at L2, GDP at L3, and GNet transport/session protocols above GDP. GNet may also be carried over other link technologies through an appropriate DLP/encapsulation profile.
