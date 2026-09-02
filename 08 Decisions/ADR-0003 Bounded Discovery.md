---
id: adr-0003-bounded-discovery
title: "ADR-0003 Bounded Discovery"
aliases: ["Decision 0003"]
type: decision
status: accepted
tags: ["gnet","gnet/decision","gnet/status/accepted"]
parent: "[[Decisions MOC]]"
related: ["[[Discovery and Bootstrap]]","[[Discovery Packets]]"]
updated: 2026-09-02
---
# Decision 0003: Generic, bounded discovery

> [!info] Knowledge graph
> **Up:** [[Decisions MOC]] · **Related:** [[Discovery and Bootstrap]] · [[Discovery Packets]]


Status: **ACCEPTED**

Use generic SOLICIT/ADVERTISE messages with registered service types. The initial router solicitation is the only inherent link-local fanout. Its responses are direct. After address configuration, directory and other discovery use scoped unicast/routed messages.

The standard startup path is router, address, directory, then selected service. This keeps ROM clients simple without turning the internet into a broadcast discovery domain.
