---
id: current-protocol-stack-summary
title: "Current Protocol Stack Summary"
type: architecture
status: active
layers: ["L1","L2","L3","L4","L5"]
tags: ["gnet","gnet/architecture","gnet/status/active"]
parent: "[[GNet Architecture Overview]]"
related: ["[[GNet PHY Profiles]]","[[GNet Link Control Protocol]]","[[Direct Link Protocol]]","[[GDP Datagram]]"]
updated: 2026-09-03
---
# Current protocol stack summary

## Native link baseline

- Minimum GNet-3 is universal.
- Four copper pairs: control up/down and data up/down.
- 3 Mbit/s nominal; 1.5 and 0.75 Mbit/s fallback.
- 32-bit flit: `VCID:2 | carried:30`.
- No SOF bit.
- GLCP handles local bootstrap, capability negotiation, credits, grants, VC allocation, abort/reset/status.
- One credit is one physical flit of guaranteed downstream capacity.

## DLP

- Minimal hop-local data transfer.
- First flit on an inactive allocated VC starts the segment.
- Hop-local integrity belongs here; exact CRC packing remains draft.
- No separate current DLP size-class registry.

## GDP

- Routed L3 package.
- 64-bit Source and Destination.
- Version, Type, Size Class, Hop Limit, QoS.
- Existing 16-value GDP package-size registry.
- No GDP checksum, CRC, Flow Control ID, session state, receive window, or fragmentation state.

## LAN products/profiles

```text
GC3  -> cheap shared 3 Mbit/s
GS3  -> independent switched 3 Mbit/s paths
GS10 -> independent ports negotiating 3/10 Mbit/s
```

No normal GC10 exists.

## Higher layers

GTS and application protocols own transport/session reliability, end-to-end integrity, tunneling, streams, security, and application semantics.
