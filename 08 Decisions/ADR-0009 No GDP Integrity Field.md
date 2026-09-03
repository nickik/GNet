---
id: adr-0009-no-gdp-integrity-field
title: "ADR-0009 No GDP Integrity Field"
aliases: ["Decision 0009","No GDP checksum"]
type: decision
status: accepted
layers: ["L2","L3","L4","L5"]
tags: ["gnet","gnet/decision","gnet/status/accepted","gnet/layer/l2","gnet/layer/l3","gnet/layer/l4","gnet/layer/l5"]
parent: "[[Decisions MOC]]"
related: ["[[GDP Protocol]]","[[GDP Datagram]]","[[Direct Link Protocol]]","[[GTS Protocol]]"]
updated: 2026-09-02
---
# Decision 0009: GDP has no integrity field

> [!info] Knowledge graph
> **Up:** [[Decisions MOC]] · **Related:** [[GDP Protocol]] · [[GDP Datagram]] · [[Direct Link Protocol]] · [[GTS Protocol]]

Status: **ACCEPTED**

## Decision

The GDP packet contains no header checksum, payload checksum, CRC, hash, integrity flag, or integrity trailer. Its only fields are Version, Type, Hop Limit, QoS, Source Address, and Destination Address.

Integrity is layered as follows:

- DLP detects corruption on one link segment and removes its integrity trailer at ingress.
- A router creates a new DLP segment and integrity value for the next link.
- Router memories and internal datapaths are implementation concerns and should use parity or ECC where required.
- GTS or another GDP-carried protocol provides end-to-end checksum, authentication, or stronger integrity when needed.

## Rationale

Hop Limit changes at every router, so a GDP header checksum would have to be recomputed at every hop. It would duplicate DLP detection for transmission errors while providing no end-to-end payload protection. Keeping integrity out of GDP preserves the minimal forwarding header and allows higher layers to choose an error-detection or authentication strength appropriate to the service.

Any end-to-end checksum or authentication construction should bind the immutable GDP fields needed to detect misdelivery, including Version, Type, Source, and Destination. Whether QoS is included depends on whether later specifications permit routers to remark it.

## Consequences

- The DLP integrity algorithm and router internal-error model must be strong enough for the target deployment.
- GDP packet diagrams must never show an integrity field or count a DLP trailer as part of GDP.
- Protocols carried directly by GDP are responsible for end-to-end integrity when silent corruption is unacceptable.
