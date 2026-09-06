---
id: gdp-datagram
title: "GDP Datagram"
aliases: ["GDP packet","GDP package"]
type: packet
status: draft
layers: ["L3"]
tags: ["gnet","gnet/packet","gnet/status/draft","gnet/layer/l3"]
parent: "[[Packet Formats MOC]]"
related: ["[[GDP Protocol]]","[[32-bit Flit Format]]","[[Direct Link Protocol]]","[[ADR-0015 Restore Minimal GDP Header]]","[[ADR-0017 32-bit Data Flit and PHY Phits]]"]
updated: 2026-09-06
---
# GDP datagram packet

Status: **FROZEN semantic field set; DRAFT exact encoding**

GDP is the Layer-3 routed datagram protocol. It contains no session, reliability, flow-control, or integrity state. Link credits belong to GLCP/DLP and transport state belongs above GDP.

## Current 20-octet encoding candidate

The current compact candidate keeps a 20-octet GDP header while preserving 64-bit addresses:

```text
    Flit 1 / Word 1
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | Ver   | Type  | Size  | Hop Limit  |      QoS      |Reserved |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

    Flits 2-3 / Words 2-3
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                       Source Address                          |
   +                                                               +
   |                         64 bits                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

    Flits 4-5 / Words 4-5
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Destination Address                        |
   +                                                               +
   |                         64 bits                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

The header is exactly **160 bits = 20 octets = five 32-bit data flits**. Baseline VC2 metadata is associated with those flits by the link layer and consumes none of the GDP header bits. The GDP payload therefore begins at the start of the sixth data flit.

The exact malformed-packet rules and Version/Type/QoS registries remain DRAFT.

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

For the current five-flit header:

```text
payload_flits = ceil(payload_bytes / 4)
GDP_content_flits = 5 + payload_flits
```

These counts exclude any DLP integrity/trailer flits. A final payload flit may be partially occupied where the payload byte count is not a multiple of four; exact padding validation belongs to the final GDP/DLP encoding rules.

| ID | Name | Payload bytes | Payload flits | GDP content flits |
|---:|---|---:|---:|---:|
| 0 | `empty` | 0 | 0 | 5 |
| 1 | `tiny3B` | 3 | 1 | 6 |
| 2 | `ctrl32B` | 32 | 8 | 13 |
| 3 | `ctrl64B` | 64 | 16 | 21 |
| 4 | `msg128B` | 128 | 32 | 37 |
| 5 | `msg256B` | 256 | 64 | 69 |
| 6 | `medium512B` | 512 | 128 | 133 |
| 7 | `bulk1K` | 1024 | 256 | 261 |
| 8 | `legacyet` | 1500 | 375 | 380 |
| 9 | `xmtu2K` | 2048 | 512 | 517 |
| 10 | `jumbo8K` | 8192 | 2048 | 2053 |
| 11 | `ultra16K` | 16384 | 4096 | 4101 |
| 12 | `mega32K` | 32768 | 8192 | 8197 |
| 13 | `giga64K` | 65536 | 16384 | 16389 |
| 14 | `jumbogram256K` | 262144 | 65536 | 65541 |
| 15 | `jumbogram1M` | 1048576 | 262144 | 262149 |

A physical/link profile MAY restrict which GDP classes it accepts. In particular GC3 deliberately excludes large/jumbo classes; see [[GNet Coupler]].

## Processing rules

- A router decrements Hop Limit before forwarding; expiry discards the packet.
- GDP itself does not fragment a package.
- Source and Destination are transmitted most-significant bit first.
- Link/profile capability constrains usable Size Classes where required.
- Header corruption is handled by hop-local DLP integrity; GDP does not add a second checksum.
