---
id: implementation-boundary
title: "Implementation Boundary"
aliases: ["QDX-GNET boundary"]
type: implementation
status: mixed
tags: ["gnet","gnet/implementation","gnet/status/mixed"]
parent: "[[Implementation MOC]]"
related: ["[[Direct Link Protocol]]","[[GNet Layer Model]]"]
updated: 2026-09-02
---
# Implementation and accelerator boundary

> [!info] Knowledge graph
> **Up:** [[Implementation MOC]] · **Related:** [[Direct Link Protocol]] · [[GNet Layer Model]]


Status: **ACCEPTED architecture; DRAFT command ABI**

GNet interoperability is defined by link and network wire protocols. PLIO and QDX are implementation mechanisms:

- **PLIO** supplies the electrical/system-I/O contract: MMIO, arbitration, DMA, discovery, interrupts, and backplane behavior. It is not the processor memory bus.
- **QDX** supplies the software-facing queued-device ABI and device profiles.
- **QDX-GNET** exposes correct basic GNet frame I/O.
- **QDX-GNETA** is an optional acceleration profile. It may reduce copying or host intervention but may not add semantics required for interoperability.

## Base QDX-GNET operations

| Operation | Purpose |
|---|---|
| POST_RX | provide a receive buffer and capacity for a port |
| TRANSMIT | transmit a frame from a DMA buffer through a port |

The completion record returns the port, buffer, actual length, status, and notification information.

## Optional accelerator operations

MULTI_TRANSMIT, RECEIVE_ANY, COPY_FRAME, BATCH_RX, BATCH_TX, and QUEUED_COMMANDS may improve switching throughput.

Hardware may also parse GDP headers, maintain queues, perform prefix or established-flow lookups, and move packet data by DMA. ROUTE, RETRANSMIT, CONGESTION_CONTROL, RPC, discovery, authentication, and name lookup remain host/protocol functions unless an accelerator is exactly behavior-compatible with the software implementation.

Integrated one-board devices remain logically PLIO/QDX devices so that small GS routers and modular backplane systems share drivers and management.
