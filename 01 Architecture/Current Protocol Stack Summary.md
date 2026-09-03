---
id: current-protocol-stack-summary
title: "Current Protocol Stack Summary"
type: architecture
status: draft
layers: ["L2","L3","L4","L5"]
tags: ["gnet","gnet/architecture","gnet/status/draft"]
parent: "[[GNet Architecture Overview]]"
related: ["[[Direct Link Protocol]]","[[32-bit Flit Format]]","[[GDP Datagram]]","[[GNet Transport]]"]
updated: 2026-09-03
---
# Current protocol stack summary

This note is the short current-state reference for the GNet protocol suite after the September 2026 layer refactor.

## Layer 2 — Direct Link Protocol (DLP)

DLP is deliberately minimal and direct-link oriented.

- 32-bit logical flit.
- `2-bit VCID | 1-bit SOF | 29 carried bits`.
- On the first flit only, the first carried bit is **Frame Type**.
- Current Frame Type values: GDP Data and Hello only.
- No network addresses or sessions.
- CRC-8 is the intended link-level corruption check; exact CRC parameters/packing remain to be frozen.
- Hello is a single-flit link-local presence message.

## Layer 3 — GDP (Global Data Protocol)

GDP is the routed datagram layer, analogous in role to IP.

- Six-flit fixed header.
- 58-bit source address and 58-bit destination address.
- First-flit fields: Type 4, Version 4, Size Class 4, Hop Limit 8, QoS 8.
- Second-flit fields: Header Checksum 8, Flow Control ID 16, Reserved 5.
- GDP Header Checksum protects the header only.
- No session/reliability state in GDP.
- No GDP fragmentation.
- 4-bit Size Class gives sixteen fixed payload sizes from 0 bytes through 1 MiB.

## Layer 4/5 — GNet

GNet is the connection/tunnel/stream layer, analogous in role to TCP but designed around a separate tunnel and connection model.

- Establishes a tunnel/connection explicitly above GDP.
- Uses a 64-bit Tunnel ID as the persistent public tunnel identity.
- Uses source/destination ports during connection establishment.
- Carries receive-window/window-scale information for reliable transport.
- Reliability, retransmission, congestion control, and final packet formats remain active design work.
- Transport integrity belongs here rather than in GDP payload routing.

## Design principle

The stack should remain modular:

```text
Applications / Services
        |
      GNet            connection, tunnel, reliability
        |
       GDP            global addressing and routing
        |
       DLP            direct-link framing and flits
        |
Physical media        serial, GNet copper/fiber, telecom PHYs, etc.
```

Other LAN technologies such as Ethernet or ARCNET can also carry GDP through an appropriate adaptation, allowing GDP/GNet to coexist with legacy physical/link networks.
