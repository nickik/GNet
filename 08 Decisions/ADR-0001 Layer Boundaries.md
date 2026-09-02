---
id: adr-0001-layer-boundaries
title: "ADR-0001 Layer Boundaries"
aliases: ["Decision 0001"]
type: decision
status: frozen
tags: ["gnet","gnet/decision","gnet/status/frozen"]
parent: "[[Decisions MOC]]"
related: ["[[GNet Layer Model]]","[[Implementation Boundary]]"]
updated: 2026-09-02
---
# Decision 0001: Keep routers below sessions

> [!info] Knowledge graph
> **Up:** [[Decisions MOC]] · **Related:** [[GNet Layer Model]] · [[Implementation Boundary]]


Status: **FROZEN**

DLP provides link framing; GDP provides routed delivery; GTS and applications provide sessions, reliability, fragmentation, encryption, and identity. This boundary keeps router hardware small, fixed-format, and independent of application evolution.

QDX may accelerate link and GDP work but does not become a protocol layer. Accounting and user authentication stay outside the forwarding fast path.
