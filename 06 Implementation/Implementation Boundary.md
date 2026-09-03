---
id: implementation-boundary
title: "Implementation Boundary"
aliases: ["QDX-GNET boundary"]
type: implementation
status: mixed
tags: ["gnet","gnet/implementation","gnet/status/mixed"]
parent: "[[Implementation MOC]]"
related: ["[[Minimum GNet-3 NIC]]","[[Direct Link Protocol]]","[[GNet Link Control Protocol]]","[[GNet Layer Model]]"]
updated: 2026-09-03
---
# Implementation and accelerator boundary

Status: **ACCEPTED architecture; DRAFT command ABI**

GNet interoperability is defined by the network/link specification. PLIO and QDX are implementation mechanisms:

- **PLIO** supplies system-I/O/backplane behavior.
- **QDX** supplies the software-facing queued-device ABI and device profiles.
- **QDX-GNET** exposes correct basic GNet packet/link I/O.
- optional acceleration may reduce copies or host intervention but may not add required private semantics.

Every native adapter still implements [[Minimum GNet-3 NIC]] compatibility before negotiating advanced link modes.

Hardware may maintain VC allocation, per-VC receive contexts, exact flit-credit counters, GLCP state, DLP integrity, queues, GDP header parsing, prefix lookup, switch output/path state, and DMA. Baseline wire parsing uses 30 carried bits per VC2 flit.

A switch/router implementation MAY use cut-through/wormhole flow with small buffers. Correctness must not require a hidden large whole-packet buffer when the GNet credit contract is satisfied.

Routing policy, transport retransmission/congestion control, RPC, directory lookup, authentication, and application semantics remain host/protocol functions unless an accelerator is exactly behavior-compatible.
