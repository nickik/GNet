---
id: gdp-datagram
title: "GDP Datagram"
aliases: ["GDP packet","GDP package"]
type: packet
status: frozen
layers: ["L3"]
tags: ["gnet","gnet/packet","gnet/status/frozen","gnet/layer/l3"]
parent: "[[Packet Formats MOC]]"
related: ["[[GDP Protocol]]","[[34-bit Flit Format]]","[[Direct Link Protocol]]","[[ADR-0018 GDP Header CRC and Local 16-bit Form]]"]
updated: 2026-09-11
---
# GDP datagram packet

Status: **FROZEN initial field semantics, size classes, address forms, and fixed-header packing**

GDP is the Layer-3 routed datagram protocol. It contains no session, reliability, flow-control, fragmentation, or payload-integrity state. Its CRC-8 protects only the GDP header information defined below.

## Address forms

The initial profile defines exactly two GDP address forms. Mixed short/full source and destination encoding is not supported.

- **Local form** — both Source and Destination are 16-bit local identifiers within the same known local GDP prefix/context.
- **Global form** — both Source and Destination are complete 64-bit GDP addresses.

A one-bit Local/Global indicator selects the form. Its fixed wire position is bit 21 of Flit 1. The numeric polarity of the Local/Global values remains governed by the Address Form registry/definition; this document freezes the field position and semantics without creating an additional mixed form.

Local GDP is non-routable beyond the local context as encoded. A router/gateway that sends traffic beyond that context constructs a new Global-form GDP packet using the canonical 64-bit source and destination identities. Likewise, when delivering global traffic onto a local context, a router may emit a new Local-form GDP packet only when both endpoints can be represented by that local context. There is no Local->Global or Global->Local mixed wire form.

## Common first-flit fields

Both address forms place the common immutable control fields and CRC in the same positions:

```text
Bits 31..30   Version        2 bits
Bits 29..26   Type           4 bits
Bits 25..22   Size Class     4 bits
Bit  21       Address Form   1 bit
Bits 20..16   Reserved       5 bits
Bits 15..8    CRC-8          8 bits
```

The CRC-8 therefore occupies the third byte of Flit 1 in both forms.

## Global form

The Global fixed header is exactly five 32-bit carried flits = 160 bits.

### Global Flit 1

```text
Bits 31..30   Version        2 bits
Bits 29..26   Type           4 bits
Bits 25..22   Size Class     4 bits
Bit  21       Address Form   1 bit = GLOBAL
Bits 20..16   Reserved       5 bits
Bits 15..8    CRC-8          8 bits
Bits 7..0     Hop Limit      8 bits
```

```text
 31                                                    0
+---------+------+-----------+------+----------+-------+-----------+
| Version | Type | SizeClass | Form | Reserved | CRC-8 | Hop Limit |
|    2    |  4   |     4     |  1   |    5     |   8   |     8     |
+---------+------+-----------+------+----------+-------+-----------+
```

### Global Flits 2-5

```text
Flit 2   Destination Address bits 63..32
Flit 3   Destination Address bits 31..0
Flit 4   Source Address bits 63..32
Flit 5   Source Address bits 31..0
```

Destination is serialized before Source so a forwarding node receives the route-selection address as early as possible.

## Local form

The Local fixed header is exactly two 32-bit carried flits = 64 bits.

### Local Flit 1

```text
Bits 31..30   Version        2 bits
Bits 29..26   Type           4 bits
Bits 25..22   Size Class     4 bits
Bit  21       Address Form   1 bit = LOCAL
Bits 20..16   Reserved       5 bits
Bits 15..8    CRC-8          8 bits
Bits 7..4     Reserved       4 bits
Bits 3..0     Hop Limit      4 bits
```

```text
 31                                                          0
+---------+------+-----------+------+----------+-------+------+-----+
| Version | Type | SizeClass | Form | Reserved | CRC-8 | Res. | Hop |
|    2    |  4   |     4     |  1   |    5     |   8   |  4   |  4  |
+---------+------+-----------+------+----------+-------+------+-----+
```

### Local Flit 2

```text
Bits 31..16   Destination ID   16 bits
Bits 15..0    Source ID        16 bits
```

The local prefix/context is known from network configuration and is not repeated in every packet. A local 16-bit identifier therefore expands to one canonical 64-bit GDP address when endpoint identity is required above GDP.

## GDP header CRC-8

GDP uses the existing `CRC-8-GNET` primitive:

```text
width       8
polynomial  0x07   (x^8 + x^2 + x + 1)
init        0x00
refin       false
refout      false
xorout      0x00
check       "123456789" -> 0xF4
```

CRC processing is most-significant bit first.

For a GDP header, the CRC input is the following canonical bitstream with no padding between fields:

```text
Version | Type | Size Class | Address Form | Destination | Source
```

Each field is processed most-significant bit first. For the multi-byte Destination and Source fields, bytes are processed in transmitted order, most-significant byte first, and each byte is processed bit 7 first through bit 0. Destination and Source are 64 bits each in Global form and 16 bits each in Local form.

The CRC explicitly excludes:

- the CRC field itself;
- Hop Limit;
- currently Reserved bits.

Hop Limit is excluded because routers modify it. A forwarding node can therefore decrement Hop Limit without recalculating CRC-8. Reserved bits are transmitted as zero in this revision and ignored on receipt; they are not part of CRC coverage. A future specification that assigns semantics to a reserved field may explicitly define whether that newly assigned field participates in CRC calculation.

CRC-8 is a GDP **header-integrity check only**. It does not protect the GDP payload and does not provide end-to-end data integrity.

A header CRC failure invalidates the entire GDP packet. The packet MUST NOT be routed or passed upward as valid.

### Invalid-header resynchronization

Current DLP does not provide an independent packet boundary that remains trustworthy when GDP Size Class itself is corrupted. Exact resynchronization after a failed GDP header CRC is therefore still an unresolved link/profile rule. A receiver MUST NOT reinterpret following carried data from the failed packet as a fresh GDP header merely because the preceding header failed validation. The mechanism that establishes the next trustworthy GDP packet boundary remains to be frozen separately; this specification does not introduce DLP CRC windows or a DLP transfer identifier to solve it.

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

| Field | Global bits | Local bits | State |
|---|---:|---:|---|
| Version | 2 | 2 | frozen |
| Type | 4 | 4 | frozen initial registry |
| Size Class | 4 | 4 | frozen |
| Address Form | 1 | 1 | frozen semantic meaning and bit position |
| Header CRC | 8 | 8 | frozen |
| Hop Limit | 8 | 4 | frozen |
| Destination | 64 | 16 | frozen |
| Source | 64 | 16 | frozen |
| QoS / Traffic Class | — | — | not present in initial profile |

GDP contains **no payload checksum, Flow Control ID, session ID, fragmentation state, option chain, or QoS field** in the initial profile.

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
| header CRC failure | drop entire packet; no forwarding; resynchronization follows the applicable link/profile rule |
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

- A router validates the GDP header CRC before treating the packet as a valid routed packet.
- A router performs destination lookup on every traditional packet-routed Global GDP package.
- Local GDP is confined to its local addressing context and does not use mixed local/global address forms.
- A router decrements Hop Limit before forwarding; expiry discards the packet. Hop Limit is not CRC covered.
- GDP itself does not fragment a package.
- Global Source and Destination are transmitted most-significant bit first, Destination before Source.
- Link/profile capability constrains usable Size Classes where required.
- GDP adds no payload or end-to-end checksum; GTS supplies end-to-end integrity for GTS payloads.
- Flow-ID/virtual-circuit based routing optimization is outside the initial traditional packet-routed profile.
