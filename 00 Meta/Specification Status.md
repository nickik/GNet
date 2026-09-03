---
id: specification-status
title: "Specification Status"
aliases: ["GNet status"]
type: meta
status: mixed
tags: ["gnet","gnet/meta","gnet/status/mixed"]
parent: "[[GNet Home]]"
related: ["[[Open Questions]]","[[GNet Architecture Overview]]","[[Decisions MOC]]"]
updated: 2026-09-03
---
# Specification status

This repository is the canonical working specification for GNet. It is coherent enough to define the baseline LAN architecture, but electrical limits and several wire encodings still require validation before independent hardware implementations should claim full conformance.

## FROZEN / ACCEPTED baseline

- DLP is the minimal hop-local data layer; GDP is the minimal routed L3 package; transport/session functions are above GDP.
- Every baseline native GNet flit is exactly 32 bits: **2-bit hop-local VCID + 30 carried bits**.
- There is **no separate SOF bit**. The first data flit received on an inactive allocated VC begins a segment implicitly.
- Baseline VCIDs provide four wire VCs and are replaced/terminated at forwarding nodes.
- Future VC4 (`4 + 28`) may be negotiated only as an advanced profile; GNet-3 and GNet-10 use VC2.
- **Minimum GNet-3** is the universal native-NIC compatibility profile. Advanced NICs begin there and negotiate upward.
- Native GNet-3/10 copper uses four balanced pairs: CONTROL-UP, CONTROL-DOWN, DATA-UP, DATA-DOWN.
- GNet-3 nominal data rate is 3 Mbit/s with mandatory 1.5 and 0.75 Mbit/s fallback modes.
- **1 credit = guaranteed downstream receive capacity for exactly one physical flit.**
- CREDIT and GRANT are distinct: receiver credit makes transmission safe; infrastructure grant schedules when reserved credit may be consumed.
- Minimum GNet-3 priority has exactly `NORMAL` and `REALTIME`.
- GC3 is the low-cost shared 3 Mbit/s Coupler; GS3 is switched 3 Mbit/s; GS10 provides independently negotiated 3/10 Mbit/s switched ports.
- There is no normal general-purpose GC10 LAN profile.
- GDP semantic fields are Version, Type, Size Class, Hop Limit, QoS, 64-bit Source, and 64-bit Destination.
- GDP contains no checksum, CRC, Flow Control ID, receive window, session ID, fragmentation state, or options.
- GDP uses the existing four-bit package-size registry from empty through 1 MiB jumbogram; link profiles may restrict usable classes.
- Hop-local accidental-error detection belongs to DLP; end-to-end integrity belongs above GDP.
- GNet uses hierarchical global addresses and does not depend on Ethernet MAC learning, collision domains, or NAT.

## ACCEPTED product/profile direction

```text
GC3-8 / GC3-16
    shared 3 Mbit/s

GS3-8 / GS3-16
    independent 3 Mbit/s switched paths

GS10-8 / GS10-16
    independent ports negotiating 3 or 10 Mbit/s
```

GNET-A remains a separate centrally scheduled residential-access family. GNET-P remains a separate point-to-point infrastructure/trunk family currently described at 10/25/50 Mbit/s.

## DRAFT / validation items

- Exact DLP CRC algorithm and trailer packing.
- Exact GLCP opcode allocation, control-word encoding, serialization, and electrical line code.
- Exact electrical limits, attenuation/crosstalk masks, reach, termination, isolation, and connector pinout for each copper grade.
- GC3-32 feasibility/economics and exact sustained-REALTIME anti-starvation rule.
- GNet-20 bonded-lane/in-band-control encoding.
- Final GNET-P control/framing and commercial name separation from LAN GNet-10.
- GDP Version/Type/QoS allocations and final validation of the 20-octet packing.
- GCTL bootstrap addressing/encodings and full routing protocol.
- GTS transport algorithms and application protocols.

## Interpretation rule

When documents conflict, the most recent accepted ADR wins, followed by this status page, then accepted protocol/profile notes, then draft notes. Historical/chat material is never normative.
