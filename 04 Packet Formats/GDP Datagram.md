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

Status: **FROZEN initial field semantics, size classes, local/global forms, and processing rules; DRAFT final bit packing**

GDP is the Layer-3 routed datagram protocol. It contains no session, reliability, flow-control, or integrity state. Link credits belong to GLCP/DLP and transport state belongs above GDP.

## Address forms

The initial profile defines exactly two GDP address forms. Mixed short/full source and destination encoding is not supported.

- **Local form** — both Source and Destination are 8-bit local identifiers within the same known local GDP prefix/context.
- **Global form** — both Source and Destination are complete 64-bit GDP addresses.

A one-bit Local/Global indicator is sufficient. The exact placement of that bit in the first protocol word remains a packing detail.

Local GDP is non-routable beyond the local context as encoded. A router/gateway that sends traffic beyond that context constructs a new Global-form GDP packet using the canonical 64-bit source and destination identities. Likewise, when delivering global traffic onto a local context, a router may emit a new Local-form GDP packet only when both endpoints can be represented by that local context. There is no Local->Global or Global->Local mixed wire form.

## Global form

The global form carries:

```text
Version              4 bits
Type                 4 bits
Size Class           4 bits
Address Form          1 bit = GLOBAL
Hop Limit             8 bits
Reserved              remaining fixed-header bits
Destination Address  64 bits
Source Address       64 bits
```

The final packing of the fixed control bits remains DRAFT. QoS/Traffic Class is not part of the initial GDP profile; any old QoS bits are reserved until a future extension defines semantics.

## Local form

The local form carries the same protocol-control semantics but both addresses are 8-bit local identifiers:

```text
Version              4 bits
Type                 4 bits
Size Class           4 bits
Address Form          1 bit = LOCAL
Hop Limit             8 bits
Reserved              remaining fixed-header bits
Destination ID        8 bits
Source ID             8 bits
```

The local prefix/context is known from the link/network configuration and is not repeated in every packet. A local 8-bit identifier therefore expands to one canonical 64-bit GDP address when endpoint identity is required above GDP.

## Frozen GDP Type registry

The initial traditional packet-routed profile uses the 4-bit Type field only to distinguish the two required GDP payload protocols:

| Value | Type | Meaning |
|---:|---|---|
| `0x0` | RESERVED | invalid/unassigned |
| `0x1` | GCTL | GNet routed control/error/diagnostic payload |
| `0x2` | GTS | GNet transport/session payload |
| `0x3`-`0xF` | RESERVED | undefined in this profile |

An undefined/reserved Type is not passed upward as another protocol. The packet is dropped and, when a valid routable source exists and GCTL rules permit, the sender is informed using the defined unsupported-payload-type error.

## Frozen GDP fields

| Field | Bits | State |
|---|---:|---|
| Version | 4 | frozen semantic width |
| Type | 4 | frozen initial registry |
| Size Class | 4 | frozen |
| Address Form | 1 | frozen semantic meaning: Local or Global only |
| Hop Limit | 8 | frozen |
| QoS / Traffic Class | — | not present in initial profile |
| Destination | 8 or 64 | both local/global forms frozen |
| Source | 8 or 64 | both local/global forms frozen |

GDP contains **no header checksum, CRC, Flow Control ID, session ID, fragmentation state, option chain, or QoS field** in the initial profile.

## GDP package size classes

The initial traditional packet-routed profile freezes this explicit four-bit size registry.

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

The 3-byte class is retained as a deliberate optimization for very small traffic. It is too small for the ordinary GTS DATA header and is not used by the initial traditional GTS format.

A physical/link profile MAY restrict which GDP classes it accepts. GDP itself does not fragment a package in transit.

## Malformed and failed packet handling

An invalid GDP packet is never forwarded as though it were valid. The receiving node/router drops it and, when a valid routable source is available and GCTL error-generation rules permit a reply, reports the failure using the defined GCTL mechanism.

Baseline mapping:

| Condition | Action / GCTL result |
|---|---|
| unsupported GDP version | drop; `PARAMETER_PROBLEM` code 0 |
| malformed header or invalid field combination | drop; `PARAMETER_PROBLEM` code 1 |
| invalid local/global address form or local identifier | drop; `PARAMETER_PROBLEM` code 3 |
| reserved/undefined GDP Type | drop; `DESTINATION_UNREACHABLE` code 2 |
| no route | drop; `DESTINATION_UNREACHABLE` code 0 |
| destination unknown/unreachable | drop; `DESTINATION_UNREACHABLE` code 1 |
| Hop Limit expires | drop; `HOP_LIMIT_EXCEEDED` code 0 |
| outgoing profile cannot carry Size Class | drop; `CLASS_UNSUPPORTED` |
| detected static routing loop / invalid route state | drop; `DESTINATION_UNREACHABLE` code 7 |
| packet accepted for forwarding but later aborted | drop; `TRANSIT_ABORTED` with the applicable reason |

Error generation is best effort and follows the GCTL anti-recursion and rate-limiting rules.

## Processing rules

- A router performs destination lookup on every traditional packet-routed Global GDP package.
- Local GDP is confined to its local addressing context and does not use mixed local/global address forms.
- A router decrements Hop Limit before forwarding; expiry discards the packet.
- GDP itself does not fragment a package.
- Global Source and Destination are transmitted most-significant bit first.
- Link/profile capability constrains usable Size Classes where required.
- GDP adds no end-to-end checksum; GTS supplies end-to-end integrity for GTS payloads.
- Flow-ID/virtual-circuit based routing optimization is outside the initial traditional packet-routed profile.
