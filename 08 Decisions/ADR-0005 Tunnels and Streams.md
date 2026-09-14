---
id: adr-0005-tunnels-and-streams
title: "ADR-0005 Tunnels and Streams"
aliases: ["Decision 0005"]
type: decision
status: accepted
layers: ["L4","L5"]
tags: ["gnet","gnet/decision","gnet/status/accepted","gnet/layer/l4","gnet/layer/l5"]
parent: "[[Decisions MOC]]"
related: ["[[GTS Protocol]]","[[GTS Transport Packets]]","[[Canonical Service Selector]]"]
updated: 2026-09-14
---
# Decision 0005: Separate tunnels, reset authority, and streams

> [!info] Knowledge graph
> **Up:** [[Decisions MOC]] · **Related:** [[GTS Protocol]] · [[GTS Transport Packets]] · [[Canonical Service Selector]]

Status: **ACCEPTED core model; later GTS specifications freeze current widths and service binding**

GTS is tunnel-first. A tunnel has a Tunnel ID and a separate Reset ID that authorizes destructive reset operations. Ordinary data omits Reset ID.

A tunnel may contain multiple streams. Transport identity and service identity are separate: Tunnel ID identifies established receiver-local transport state, Stream ID identifies one stream inside that tunnel, and [[Canonical Service Selector]] identifies the logical service selected when the tunnel is created.

The current frozen GTS profile narrows the earlier exploratory model in this ADR:

- Tunnel ID is 32 bits and receiver-local;
- Reset ID is 32 bits and receiver-local;
- Stream ID is 8 bits and tunnel-scoped;
- CONNECT selects exactly one CSS and creates Stream 0;
- the accepted tunnel remains bound to that service for its lifetime;
- STREAM_OPEN creates additional streams inside the same service-bound tunnel and does not select another service;
- selecting another service requires another tunnel.

The earlier working proposal of a 64-bit Reset ID, 16-bit Stream ID, Stream 0 as a separate control stream, and per-stream service/profile selection is historical and is superseded by the current GTS wire specification.
