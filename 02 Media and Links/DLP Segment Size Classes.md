---
id: dlp-segment-size-classes
title: "DLP Segment Size Classes"
aliases: ["GNet package sizes", "Segment classes", "Package size classes"]
type: protocol
status: accepted
layers: ["L2"]
tags: ["gnet", "gnet/protocol", "gnet/status/accepted", "gnet/layer/l2"]
parent: "[[Media and Links MOC]]"
related: ["[[Direct Link Protocol]]", "[[Virtual Channels and VCIDs]]", "[[32-bit Flit Format]]", "[[ADR-0010 DLP Segment Size Classes]]"]
updated: 2026-09-03
---
# DLP segment size classes

> [!info] Knowledge graph
> **Up:** [[Media and Links MOC]] · **Related:** [[Direct Link Protocol]] · [[Virtual Channels and VCIDs]] · [[32-bit Flit Format]] · [[ADR-0010 DLP Segment Size Classes]]

Status: **ACCEPTED**

Every DLP segment declares a two-bit Segment Class in its first flit. The class is a maximum meaningful-payload size, not an implied padding size. Payload Length still gives the exact octet count.

```text
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |   Protocol    |LC |SC |    Payload Length (octets)    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

`LC` is the two-bit Link Class. `SC` is the two-bit Segment Class.

## Class assignments

| SC | Name | Maximum meaningful payload | Maximum payload flits | Complete segment flits |
|---:|---|---:|---:|---:|
| `00` | Small | 64 octets | 19 | 21 |
| `01` | Normal | 256 octets | 74 | 76 |
| `10` | Large | 1,024 octets | 293 | 295 |
| `11` | Reserved | — | — | — |

Payload-flit counts use `ceil((8 × payload_octets) / 28)`. Complete-segment counts include one header flit and one DLP integrity-trailer flit.

## Use

- **Small** is for link control, short discovery/configuration messages, acknowledgements, and latency-sensitive work.
- **Normal** is the ordinary data class and the normal bounded-VCID working unit.
- **Large** is for efficient bulk transfer where the scheduler can safely hold a VCID longer.

The link may interleave flits from other VCIDs while a segment is active. The size class therefore reserves a bounded amount of per-VCID receiver and scheduler state; it does not grant exclusive ownership of the link.

The base network-card receive guarantee and credit-grant policy are specified separately from these maximum classes. A receiver need not accept a complete Normal or Large segment in one burst.
