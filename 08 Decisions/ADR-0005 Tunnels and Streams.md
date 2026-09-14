---
id: adr-0005-tunnels-and-streams
title: "ADR-0005 Tunnels and Streams"
aliases: ["Decision 0005"]
type: decision
status: accepted
layers: ["L4","L5"]
tags: ["gnet","gnet/decision","gnet/status/accepted","gnet/layer/l4","gnet/layer/l5"]
parent: "[[Decisions MOC]]"
related: ["[[GTS Protocol]]","[[GTS Transport Packets]]","[[GTS Stream Profiles]]","[[Canonical Service Selector]]"]
updated: 2026-09-14
---
# Decision 0005: Separate tunnels, reset authority, and streams

Status: **ACCEPTED core model; current GTS specifications freeze widths, service binding, and stream-profile behavior**

GTS is tunnel-first. A tunnel has a receiver-local Tunnel ID and separate Reset ID for destructive tunnel RESET. Ordinary data omits Reset ID.

A tunnel may contain multiple streams. Transport identity and service identity are separate: Tunnel ID identifies receiver-local tunnel state, Stream ID identifies one stream inside that tunnel, and [[Canonical Service Selector]] identifies the logical service selected by CONNECT.

The current frozen profile specifies:

- Tunnel ID: 32 bits, receiver-local;
- Reset ID: 32 bits, receiver-local;
- Stream ID: 8 bits, tunnel-scoped;
- CONNECT selects exactly one CSS and creates Stream 0;
- the tunnel remains bound to that CSS for its lifetime;
- STREAM_OPEN creates more streams inside the same tunnel;
- each stream carries a 16-bit Stream Profile defining reliability, fixed/variable sizing, optional unreliable sequencing, optional unchecked payload, direction, and Size Class;
- GTS is message-preserving: each DATA/DATA_END/DATAGRAM packet is one application message unit;
- reliable streams preserve order and message boundaries and use ACK/retransmission/receive credit;
- unreliable streams use DATAGRAM without ACK/retransmission and may optionally carry Sequence;
- unreliable streams may exclude application payload from CRC coverage while retaining CRC-32 over transport metadata;
- streams may be opener-to-peer, peer-to-opener, or bidirectional;
- STREAM_RESET / STREAM_RESET_ACK retire one stream without destroying the tunnel;
- selecting another service requires another tunnel.

Tunnel RESET remains distinct from STREAM_RESET. Tunnel RESET destroys the whole association and all streams; STREAM_RESET affects only one Stream ID.

The earlier working proposal of a 64-bit Reset ID, 16-bit Stream ID, Stream 0 as a separate control stream, and per-stream service selection is historical and superseded by the current GTS wire specification.
