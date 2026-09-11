---
id: specification-status
title: "Specification Status"
aliases: ["GNet status"]
type: meta
status: mixed
tags: ["gnet","gnet/meta","gnet/status/mixed"]
parent: "[[GNet Home]]"
related: ["[[Open Questions]]","[[GNet Architecture Overview]]","[[Decisions MOC]]"]
updated: 2026-09-11
---
# Specification status

This repository is the canonical working specification for GNet. It is coherent enough to define the baseline LAN architecture, but electrical limits and several wire encodings still require validation before independent hardware implementations should claim full conformance.

## FROZEN / ACCEPTED baseline

- DLP is the minimal hop-local data layer; GDP is the minimal routed L3 package; transport/session functions are above GDP.
- Every baseline native GNet flit is exactly 34 bits: **2-bit hop-local VCID + 32 carried bits**.
- There is **no separate SOF bit**. The first data flit received on an inactive allocated VC begins a segment implicitly.
- Baseline VCIDs provide four wire VCs and are replaced/terminated at forwarding nodes.
- Future VC4 (`4 + 32`, 36 bits total) may be negotiated only as an advanced profile; GNet-3 and GNet-10 use VC2.
- **Minimum GNet-3** is the universal native-NIC compatibility profile. Advanced NICs begin there and negotiate upward.
- Native GNet-3/10 copper uses four balanced pairs: CONTROL-UP, CONTROL-DOWN, DATA-UP, DATA-DOWN.
- GNet-3 nominal data rate is 3 Mbit/s with mandatory 1.5 and 0.75 Mbit/s fallback modes.
- GLCP 0.1 `HELLO` and `CAPABILITIES` use accepted 32-bit logical control flits on dedicated pairs; their electrical serialization remains open.
- **1 credit = guaranteed downstream receive capacity for exactly one physical flit.**
- CREDIT and GRANT are distinct: receiver credit makes transmission safe; infrastructure grant schedules when reserved credit may be consumed.
- Minimum GNet-3 priority has exactly `NORMAL` and `REALTIME`.
- GC3 is the low-cost shared 3 Mbit/s Coupler; GS3 is switched 3 Mbit/s; GS10 provides independently negotiated 3/10 Mbit/s switched ports.
- There is no normal general-purpose GC10 LAN profile.
- GDP uses 2-bit Version, 4-bit Type, 4-bit Size Class, one Address Form bit, and header-only CRC-8. Global form has 64-bit Destination and Source with an 8-bit Hop Limit; Local form has 16-bit Destination and Source IDs with a 4-bit Hop Limit.
- GDP CRC-8 protects Version, Type, Size Class, Address Form, Destination, and Source. Hop Limit, Reserved bits, and payload are not covered.
- GDP contains no payload checksum, Flow Control ID, receive window, session ID, fragmentation state, options, or QoS field.
- GDP uses the frozen 16-value package-size registry from 0 bytes through 8192 bytes; link profiles may restrict usable classes.
- The current baseline does not define periodic DLP payload-integrity windows. End-to-end payload integrity belongs above GDP.
- GNet uses hierarchical global addresses and does not depend on Ethernet MAC learning, collision domains, or NAT.
- `FE80::/16` is the reserved non-routable link-local GDP prefix; clients generate their own 48-bit suffixes.

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

- GDP invalid-header resynchronization when Size Class itself cannot be trusted; the specification must establish the next trustworthy GDP boundary without inventing periodic DLP CRC windows.
- GLCP encodings for RELEASE/ABORT, control serialization, and electrical line code.
- Exact electrical limits, attenuation/crosstalk masks, reach, termination, isolation, and connector pinout for each copper grade.
- GC3-32 feasibility/economics and exact sustained-REALTIME anti-starvation rule.
- GNet-20 bonded-lane/in-band-control encoding.
- Final GNET-P control/framing and commercial name separation from LAN GNet-10.
- GCTL bootstrap addressing/encodings and full routing protocol.
- GTS transport algorithms and application protocols.

## Interpretation rule

When documents conflict, the most recent accepted ADR wins, followed by this status page, then accepted protocol/profile notes, then draft notes. Historical/chat material is never normative.
