---
id: adr-0010-dlp-segment-size-classes
title: "ADR-0010 DLP Segment Size Classes"
aliases: ["Decision 0010", "Package size classes"]
type: decision
status: accepted
layers: ["L2"]
tags: ["gnet", "gnet/decision", "gnet/status/accepted", "gnet/layer/l2"]
parent: "[[Decisions MOC]]"
related: ["[[DLP Segment Size Classes]]", "[[Direct Link Protocol]]", "[[Virtual Channels and VCIDs]]"]
updated: 2026-09-03
---
# Decision 0010: Use bounded DLP segment size classes

> [!info] Knowledge graph
> **Up:** [[Decisions MOC]] · **Related:** [[DLP Segment Size Classes]] · [[Direct Link Protocol]] · [[Virtual Channels and VCIDs]]

Status: **ACCEPTED**

## Decision

The two-bit DLP Segment Class field defines three bounded maximum payload classes:

- `00` Small: 64 octets;
- `01` Normal: 256 octets;
- `10` Large: 1,024 octets;
- `11` reserved.

The 16-bit Payload Length remains authoritative for the exact meaningful octet count and MUST NOT exceed the selected class limit.

## Rationale

VCIDs are temporary link resources. Size classes let the transmitter, receiver, and scheduler know the maximum state and holding time before the complete payload arrives, while still allowing a final short segment. They bound wormhole occupancy without forcing a universal fixed packet size.

The class does not determine the transmission burst. Flits may be interleaved by VCID and paced by link credits or grants.
