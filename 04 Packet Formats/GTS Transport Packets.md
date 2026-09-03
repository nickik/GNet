---
id: gts-transport-packets
title: "GTS Transport Packets"
aliases: ["GTS packets"]
type: packet
status: open
layers: ["L4","L5"]
tags: ["gnet","gnet/packet","gnet/status/open","gnet/layer/l4","gnet/layer/l5"]
parent: "[[Packet Formats MOC]]"
related: ["[[GTS Protocol]]","[[ADR-0005 Tunnels and Streams]]","[[GDP Datagram]]"]
updated: 2026-09-03
---
# GTS transport packets

Status: **OPEN; semantic requirements retained, historical wire packing rejected**

The tunnel/stream model requires encodings for:

- `CONNECT` — propose Tunnel ID/reset authority/service/security/profile and initial transport flow control;
- `CONNECT_ACK` — accept and return negotiated state/handle;
- `STREAM_OPEN` / `STREAM_ACCEPT` — establish stream IDs and delivery profiles;
- `DATA` / `ACK` — carry stream data plus only profile-required sequence/ack/window state;
- `STREAM_CLOSE` — close one stream;
- `TUNNEL_CLOSE`, `RESET`, `REBIND` — destructive operations that prove reset authority.

Normal DATA MUST NOT contain Reset ID. Link `CREDIT`/`GRANT` are not GTS fields; they are GLCP/DLP hop-local state.

Earlier CONNECT/CONNECT_ACK diagrams were exploratory and assumed obsolete 4-bit-VCID/28-carried-bit DLP packing. They are not current wire encodings and are intentionally not reproduced as normative diagrams here.

The next transport decision must define CONNECT, CONNECT_ACK, DATA, ACK, RESET, and CLOSE together with exact state machines, retransmission/congestion behavior, end-to-end integrity, and golden vectors. Physical carriage then uses the current baseline VC2 DLP format without transport-specific flit alignment.
