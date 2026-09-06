---
id: current-protocol-stack-summary
title: "Current Protocol Stack Summary"
type: architecture
status: active
layers: ["L1","L2","L3","L4","L5"]
tags: ["gnet","gnet/architecture","gnet/status/active"]
parent: "[[GNet Architecture Overview]]"
related: ["[[GNet PHY Profiles]]","[[GNet Link Control Protocol]]","[[Direct Link Protocol]]","[[GDP Datagram]]"]
updated: 2026-09-06
---
# Current protocol stack summary

## Native link baseline

- Minimum GNet-3 is universal.
- Four copper pairs: control up/down and data up/down.
- 3 Mbit/s flit data nominal; 1.5 and 0.75 Mbit/s fallback.
- Flit data width: exactly 32 bits.
- Baseline VC2: 2-bit hop-local VCID associated with every flit, outside the 32 data bits.
- Physical transfer units are PHY-specific phits; an inline serial VC2 mapping carries 34 logical link bits per flit.
- No SOF bit.
- GLCP handles local bootstrap, capability negotiation, credits, grants, VC allocation, abort/reset/status.
- One credit is receive capacity for one complete 32-bit flit plus its associated link metadata, independent of phit count.

## DLP

- Minimal hop-local data transfer.
- First flit on an inactive allocated VC starts the segment.
- Hop-local integrity belongs here; exact CRC packing remains draft.
- No separate current DLP size-class registry.

## GDP

- Routed L3 package.
- 64-bit Source and Destination.
- Version, Type, Size Class, Hop Limit, QoS.
- Current header: 20 octets = exactly five 32-bit flits.
- Existing 16-value GDP package-size registry.
- No GDP checksum, CRC, Flow Control ID, session state, receive window, or fragmentation state.

## LAN products/profiles

```text
GC3  -> cheap shared 3 Mbit/s flit data
GS3  -> independent switched 3 Mbit/s flit-data paths
GS10 -> independent ports negotiating 3/10 Mbit/s flit data
```

No normal GC10 exists.

## Higher layers

GTS and application protocols own transport/session reliability, end-to-end integrity, tunneling, streams, security, and application semantics.
