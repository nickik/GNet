---
id: gdp-datagram
title: "GDP Datagram"
aliases: ["GDP packet","GDP package"]
type: packet
status: draft
layers: ["L3"]
tags: ["gnet","gnet/packet","gnet/status/draft","gnet/layer/l3"]
parent: "[[Packet Formats MOC]]"
related: ["[[GDP Protocol]]","[[34-bit Flit Format]]","[[Direct Link Protocol]]","[[ADR-0015 Restore Minimal GDP Header]]"]
updated: 2026-09-10
---
# GDP datagram packet

Status: **FROZEN initial size-class registry and processing rules; DRAFT exact local/global encoding**

GDP is the Layer-3 routed datagram protocol. It contains no session, reliability, flow-control, or integrity state. Link credits belong to GLCP/DLP and transport state belongs above GDP.

## Current global 20-octet encoding candidate

The current global candidate keeps a 20-octet GDP header while preserving 64-bit addresses:

```text
    Word 1 — logical protocol word, not a physical flit
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | Ver   | Type  | Size  | Hop Limit  |      QoS      |Reserved |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

    Words 2-3
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Destination Address                        |
   +                                                               +
   |                         64 bits                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

    Words 4-5
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                       Source Address                          |
   +                                                               +
   |                         64 bits                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

This exact first-word packing remains under review because the initial profile has not yet frozen the role of the current `Type` and `QoS` fields or the compact/local address-mode encoding. The 64-bit global source/destination semantics and four-bit Size Class are retained.

## Current fields under review

| Field | Bits | State |
|---|---:|---|
| Version | 4 | retained |
| Type | 4 | purpose under review for initial profile |
| Size Class | 4 | **frozen** |
| Hop Limit | 8 | **frozen** |
| QoS | 8 | not accepted for the initial profile unless explicit semantics are defined |
| Reserved | 4 | draft packing reserve |
| Destination Address | 64 | **frozen global form** |
| Source Address | 64 | **frozen global form** |

GDP contains **no header checksum, CRC, Flow Control ID, session ID, fragmentation state, or option chain**.

## GDP package size classes

The initial traditional packet-routed profile freezes this explicit four-bit size registry. It is deliberately concentrated below 2 KiB and provides only two jumbo classes.

| ID | Name | Payload bytes |
|---:|---|---:|
| 0 | `empty` | 0 |
| 1 | `tiny3B` | 3 |
| 2 | `ctrl32B` | 32 |
| 3 | `ctrl64B` | 64 |
| 4 | `msg128B` | 128 |
| 5 | `msg192B` | 192 |
| 6 | `msg256B` | 256 |
| 7 | `msg384B` | 384 |
| 8 | `medium512B` | 512 |
| 9 | `medium768B` | 768 |
| 10 | `bulk1K` | 1024 |
| 11 | `bulk1280B` | 1280 |
| 12 | `legacyet` | 1500 |
| 13 | `xmtu2K` | 2048 |
| 14 | `jumbo4K` | 4096 |
| 15 | `jumbo8K` | 8192 |

The 3-byte class is retained as a deliberate optimization for very small traffic such as packet voice and compact local exchanges. It is too small to carry the ordinary uncompressed GTS header defined by the current 32-bit Tunnel ID / 8-bit Stream ID direction; its use by GTS would require a separate compact form and is not part of the initial traditional GTS profile.

A physical/link profile MAY restrict which GDP classes it accepts. GDP itself does not fragment a package in transit.

## Malformed and failed packet handling

An invalid GDP packet is never forwarded as though it were valid. The receiving node/router drops it and, when a valid routable source is available and GCTL error-generation rules permit a reply, reports the failure using the defined GCTL mechanism.

Baseline mapping:

| Condition | Action / GCTL result |
|---|---|
| unsupported GDP version | drop; `PARAMETER_PROBLEM` code 0 |
| malformed header or invalid field combination | drop; `PARAMETER_PROBLEM` code 1 |
| invalid address representation | drop; `PARAMETER_PROBLEM` code 3 |
| unsupported GDP payload type, if Type remains in the final format | drop; `DESTINATION_UNREACHABLE` code 2 |
| no route | drop; `DESTINATION_UNREACHABLE` code 0 |
| destination unknown/unreachable | drop; `DESTINATION_UNREACHABLE` code 1 |
| Hop Limit expires | drop; `HOP_LIMIT_EXCEEDED` code 0 |
| outgoing profile cannot carry Size Class | drop; `CLASS_UNSUPPORTED` |
| detected static routing loop / invalid route state | drop; `DESTINATION_UNREACHABLE` code 7 |
| packet accepted for forwarding but later aborted | drop; `TRANSIT_ABORTED` with the applicable reason |

Error generation is best effort and follows the GCTL anti-recursion and rate-limiting rules. A failure to return an error never implies successful delivery.

## Processing rules

- A router performs destination lookup on every traditional packet-routed GDP package.
- A router decrements Hop Limit before forwarding; expiry discards the packet.
- GDP itself does not fragment a package.
- Global Source and Destination are transmitted most-significant bit first.
- Link/profile capability constrains usable Size Classes where required.
- GDP adds no end-to-end checksum; GTS supplies end-to-end integrity for GTS payloads.
- Flow-ID/virtual-circuit based routing optimization is outside the initial traditional packet-routed profile.
