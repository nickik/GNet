---
id: gdp-datagram
title: "GDP Datagram"
aliases: ["GDP packet","GDP package"]
type: packet
status: draft
layers: ["L3"]
tags: ["gnet","gnet/packet","gnet/status/draft","gnet/layer/l3"]
parent: "[[Packet Formats MOC]]"
related: ["[[GDP Protocol]]","[[32-bit Flit Format]]","[[Direct Link Protocol]]","[[ADR-0015 Restore Minimal GDP Header]]"]
updated: 2026-09-03
---
# GDP datagram packet

Status: **FROZEN semantic field set; DRAFT exact encoding**

GDP is the Layer-3 routed datagram protocol. It contains no session, reliability, flow-control, or integrity state. Link credits belong to GLCP/DLP and transport state belongs above GDP.

## Current 20-octet encoding candidate

The current compact candidate keeps a 20-octet GDP header while preserving 64-bit addresses:

```text
    Word 1 — logical protocol word, not a physical flit
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | Ver   | Type  | Size  | Hop Limit  |      QoS      |Reserved |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

    Words 2-3
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                       Source Address                          |
   +                                                               +
   |                         64 bits                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

    Words 4-5
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Destination Address                        |
   +                                                               +
   |                         64 bits                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

Fields are packed continuously across the baseline 30 carried bits of successive flits. A 160-bit header therefore consumes six VC2 flits and leaves 20 carried bits in the sixth flit for the beginning of the GDP payload. The exact malformed-packet rules and Version/Type/QoS registries remain DRAFT.

## Fields

| Field | Bits | Meaning |
|---|---:|---|
| Version | 4 | GDP wire version. |
| Type | 4 | GDP payload/protocol type. |
| Size Class | 4 | Selects one fixed GDP payload size. |
| Hop Limit | 8 | Decremented at every GDP router. |
| QoS | 8 | Network forwarding/service marking subject to policy. |
| Reserved | 4 | Wire-format reserve; transmit zero, ignore on receive. |
| Source Address | 64 | Hierarchical source GDP address. |
| Destination Address | 64 | Hierarchical destination GDP address. |

GDP contains **no header checksum, CRC, Flow Control ID, session ID, fragmentation state, or option chain**.

## GDP package size classes

This is the current four-bit GDP package-size registry. It replaces the superseded DLP Segment Class scheme.

| ID | Name | Payload bytes |
|---:|---|---:|
| 0 | `empty` | 0 |
| 1 | `tiny3B` | 3 |
| 2 | `ctrl32B` | 32 |
| 3 | `ctrl64B` | 64 |
| 4 | `msg128B` | 128 |
| 5 | `msg256B` | 256 |
| 6 | `medium512B` | 512 |
| 7 | `bulk1K` | 1024 |
| 8 | `legacyet` | 1500 |
| 9 | `xmtu2K` | 2048 |
| 10 | `jumbo8K` | 8192 |
| 11 | `ultra16K` | 16384 |
| 12 | `mega32K` | 32768 |
| 13 | `giga64K` | 65536 |
| 14 | `jumbogram256K` | 262144 |
| 15 | `jumbogram1M` | 1048576 |

A physical/link profile MAY restrict which GDP classes it accepts. In particular GC3 deliberately excludes large/jumbo classes; see [[GNet Coupler]].

## Processing rules

- A router decrements Hop Limit before forwarding; expiry discards the packet.
- GDP itself does not fragment a package.
- Source and Destination are transmitted most-significant bit first.
- Link/profile capability constrains usable Size Classes where required.
- Header corruption is handled by hop-local DLP integrity; GDP does not add a second checksum.
