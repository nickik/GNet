---
id: specification-status
title: "Specification Status"
aliases: ["GNet status"]
type: meta
status: mixed
tags: ["gnet","gnet/meta","gnet/status/mixed"]
parent: "[[GNet Home]]"
related: ["[[Open Questions]]","[[GNet Architecture Overview]]","[[Decisions MOC]]"]
updated: 2026-09-06
---
# Specification status

This repository is the canonical working specification for GNet. It is coherent enough to define the baseline LAN architecture, but electrical limits and several wire encodings still require validation before independent hardware implementations should claim full conformance.

## FROZEN / ACCEPTED baseline

- DLP is the minimal hop-local data layer; GDP is the minimal routed L3 package; transport/session functions are above GDP.
- One GNet **flit carries exactly 32 data bits**.
- Baseline VC2 associates a **2-bit hop-local VCID** with every flit without consuming any of the 32 flit data bits. Baseline links therefore provide four wire VCs.
- The physical transfer unit is the **phit**. Phit width, serialization, line coding, and the mapping of a 32-bit flit plus VC metadata onto phits are PHY-specific.
- A PHY that serializes baseline VC2 inline carries 34 logical link bits for every 32-bit data flit. This 34-bit inline representation is not a universal physical bus width.
- There is **no separate SOF bit**. The first data flit received on an inactive allocated VC begins a segment implicitly.
- VC state is replaced/terminated at forwarding nodes.
- VC4 or VC8 may be considered only as future negotiated wider-VC options; neither is a baseline compatibility profile, and neither changes the 32-bit flit data width.
- **Minimum GNet-3** is the universal native-NIC compatibility profile. Advanced NICs begin there and negotiate upward.
- Native GNet-3/10 copper uses four balanced pairs: CONTROL-UP, CONTROL-DOWN, DATA-UP, DATA-DOWN.
- GNet-3 nominal data rate is 3 Mbit/s of flit data with mandatory 1.5 and 0.75 Mbit/s flit-data fallback modes. GNet-10 provides 10 Mbit/s of flit data. PHY symbol/line rate includes VC metadata and any coding overhead.
- **1 credit = guaranteed downstream receive capacity for exactly one complete 32-bit flit and its associated link metadata**, independent of the number of phits used by the PHY.
- CREDIT and GRANT are distinct: receiver credit makes transmission safe; infrastructure grant schedules when reserved credit may be consumed.
- Minimum GNet-3 priority has exactly `NORMAL` and `REALTIME`.
- GC3 is the low-cost shared 3 Mbit/s Coupler; GS3 is switched 3 Mbit/s; GS10 provides independently negotiated 3/10 Mbit/s switched ports.
- There is no normal general-purpose GC10 LAN profile.
- GDP semantic fields are Version, Type, Size Class, Hop Limit, QoS, 64-bit Source, and 64-bit Destination.
- The current 20-octet GDP header occupies exactly **five 32-bit flits** before payload data.
- GDP contains no checksum, CRC, Flow Control ID, receive window, session ID, fragmentation state, or options.
- GDP uses the existing four-bit package-size registry from empty through 1 MiB jumbogram; link profiles may restrict usable classes.
- Hop-local accidental-error detection belongs to DLP; end-to-end integrity belongs above GDP.
- GNet uses hierarchical global addresses and does not depend on Ethernet MAC learning, collision domains, or NAT.

## ACCEPTED product/profile direction

```text
GC3-8 / GC3-16
    shared 3 Mbit/s flit data

GS3-8 / GS3-16
    independent 3 Mbit/s switched flit-data paths

GS10-8 / GS10-16
    independent ports negotiating 3 or 10 Mbit/s flit data
```

GNet Broadband Access remains a separate centrally scheduled residential-access family. GNet Carrier Trunk remains a separate point-to-point infrastructure/trunk family.

## DRAFT / validation items

- Exact DLP CRC algorithm and trailer packing, including any final partially occupied flit.
- Exact GLCP opcode allocation, control-word encoding, serialization, and electrical line code.
- Exact PHY mapping of 32-bit flits plus associated VC metadata onto physical phits, including framing/coding overhead.
- Exact electrical limits, attenuation/crosstalk masks, reach, termination, isolation, and connector pinout for each copper grade.
- GC3-32 feasibility/economics and exact sustained-REALTIME anti-starvation rule.
- GNet-20 bonded-lane/in-band-control encoding.
- Final GCT control/framing details.
- GDP Version/Type/QoS allocations and final malformed-packet rules for the 20-octet packing.
- GCTL bootstrap addressing/encodings and full routing protocol.
- GTS transport algorithms and application protocols.

## Interpretation rule

When documents conflict, the most recent accepted ADR wins, followed by this status page, then accepted protocol/profile notes, then draft notes. Historical/chat material is never normative.
