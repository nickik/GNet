---
id: gnet-layer-model
title: "GNet Layer Model"
aliases: ["Layer model"]
type: architecture
status: frozen
layers: ["L1","L2","L3","L4","L5","L6","L7"]
tags: ["gnet","gnet/architecture","gnet/status/frozen"]
parent: "[[Architecture MOC]]"
related: ["[[GNet Architecture Overview]]","[[Virtual Channels and VCIDs]]","[[GNet Link Control Protocol]]"]
updated: 2026-09-03
---
# GNet layer model

Status: **FROZEN architecture; selected encodings remain DRAFT**

| Layer | GNet component | Responsibilities | Explicit exclusions |
|---|---|---|---|
| L1 | GNet PHY / GLCP / GNET-A / GNET-P | signaling, clocks, pair/channel use, capability/rate negotiation, local transmission grants | global addressing, sessions, names |
| L2 | DLP | VC2 baseline, 30 carried bits, bounded hop transfer, link integrity, receiver-credit accounting, protocol adaptation | global addresses, end-to-end sessions, directory lookup |
| L3 | GDP / GCTL | 64-bit source/destination, Size Class, hop limit, QoS, routing and network control | checksum/CRC, transport flow ID/window, reliability, fragmentation state |
| L4 | GTS transport | ports, sequencing, acknowledgement, retransmission, receive flow control, fragmentation/reassembly | routing decisions, user identity |
| L5 | GTS tunnel control / GNET-S | tunnel establishment, reset/rebind authority, stream profiles, reservations, cryptographic association | application naming |
| L6 | Directory and identity | names, service discovery, credentials, authorization assertions | packet forwarding |
| L7 | GSC, GTerm, boot, file, RPC, voice control | application semantics and named-service selection | physical/link behavior |

The dedicated GNet copper control channel is GLCP, not GDP/GCTL. Link `CREDIT` is real physical-flit capacity; `GRANT` is infrastructure scheduling permission.

A forwarding node terminates incoming hop-local VC state, reads GDP, chooses the outgoing link, obtains downstream credit/path availability, and allocates new outgoing VC state. End-to-end transport state is not required in the ordinary router fast path.
