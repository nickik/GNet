---
id: gnet-layer-model
title: "GNet Layer Model"
aliases: ["Layer model"]
type: architecture
status: frozen
layers: ["L1","L2","L3","L4","L5","L6","L7"]
tags: ["gnet","gnet/architecture","gnet/status/frozen","gnet/layer/l1","gnet/layer/l2","gnet/layer/l3","gnet/layer/l4","gnet/layer/l5","gnet/layer/l6","gnet/layer/l7"]
parent: "[[Architecture MOC]]"
related: ["[[GNet Architecture Overview]]","[[ADR-0001 Layer Boundaries]]"]
updated: 2026-09-02
---
# GNet layer model

> [!info] Knowledge graph
> **Up:** [[Architecture MOC]] · **Related:** [[GNet Architecture Overview]] · [[ADR-0001 Layer Boundaries]]


Status: **FROZEN architecture; DRAFT protocol names**

| Layer | GNet component | Responsibilities | Explicit exclusions |
|---|---|---|---|
| L1 | GNET-L / GNET-A / GNET-P media | signaling, clocks, pair/channel use | addressing, routing, names |
| L2 | DLP / GNET-LINK | framing, local endpoint/port delivery, local arbitration, CRC-8, link protocol demultiplexing | global addresses, sessions, directory lookup |
| L3 | GDP | 64-bit source/destination, hop limit, QoS marking, next-protocol type | length, checksum, fragmentation, options, reliability, session/flow ID |
| L4 | GTS transport | ports, sequencing, acknowledgment, retransmission, flow control, fragmentation/reassembly | routing decisions, user identity |
| L5 | GTS tunnel control / GNET-S | tunnel establishment, reset/rebind authority, stream profiles, reservations, mobility binding, cryptographic association | application naming |
| L6 | Directory and identity services | names, service discovery, credentials, authorization assertions | packet forwarding |
| L7 | GSC, GTerm, boot, file, RPC, voice control | application semantics and named-service selection | physical/link behavior |

DLP is protocol-neutral and may carry GDP, DECnet, TCP/IP, XNS, SNA gateway traffic, diagnostics, or link-local GCTL.

## Forwarding path

A router validates the incoming DLP frame, removes L2 framing, reads the fixed GDP header, decrements Hop Limit, chooses the next link from the destination prefix and QoS, and emits a new DLP frame. It does not inspect transport state in the ordinary forwarding path.

## QDX boundary

QDX-GNET hardware may perform DLP framing, CRC, queues, GDP header parsing, prefix lookup, and notification/DMA. It MUST NOT redefine GDP addressing, directory behavior, authentication, or end-to-end transport. This keeps a one-board GS switch and a large QDX-backplane switch interoperable.
