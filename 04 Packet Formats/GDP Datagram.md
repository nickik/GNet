---
id: gdp-datagram
title: "GDP Datagram"
aliases: ["GDP packet"]
type: packet
status: draft
layers: ["L3"]
tags: ["gnet","gnet/packet","gnet/status/draft","gnet/layer/l3"]
parent: "[[Packet Formats MOC]]"
related: ["[[GDP Protocol]]","[[32-bit Flit Format]]","[[Direct Link Protocol]]"]
updated: 2026-09-03
---
# GDP datagram packet

> [!info] Knowledge graph
> **Up:** [[Packet Formats MOC]] · **Related:** [[GDP Protocol]] · [[32-bit Flit Format]] · [[Direct Link Protocol]]

Status: **CURRENT DRAFT**

GDP is the Layer-3 routed datagram protocol. Transport sessions, reliability, ports, tunnel state, and end-to-end payload integrity are not GDP responsibilities; those belong to GNet or another protocol carried as GDP payload.

## Six-flit GDP header

Every GDP packet begins with exactly six 32-bit flits. The first flit uses the DLP first-flit Frame Type bit. Continuation flits use all 29 carried bits.

```text
    Flit 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |VCID |S|F| Type  | Ver   | Size  | Hop Limit  |      QoS      |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

    Flit 2
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |VCID |S| Header Checksum |   Flow Control ID   | Reserved     |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

    Flit 3
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |VCID |S|            Source Address [57:29]                    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

    Flit 4
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |VCID |S|            Source Address [28:0]                     |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

    Flit 5
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |VCID |S|          Destination Address [57:29]                 |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

    Flit 6
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |VCID |S|          Destination Address [28:0]                  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

Where:

- **VCID:** 2 bits in every flit.
- **SOF (`S`):** 1 only on Flit 1; 0 on Flits 2–6 and payload continuation flits.
- **Frame Type (`F`):** 1 bit in the first carried position of Flit 1. `0` means GDP data; `1` means Hello.

## GDP fields

| Field | Bits | Meaning |
|---|---:|---|
| Frame Type | 1 | DLP first-flit discriminator; GDP data is `0`. |
| Type | 4 | GDP payload/protocol type registry. |
| Version | 4 | GDP protocol version. |
| Size Class | 4 | Selects one of 16 fixed payload sizes. |
| Hop Limit | 8 | Decremented at every GDP router; prevents loops. |
| QoS | 8 | Packet priority/service class. |
| Header Checksum | 8 | Lightweight checksum over the GDP header. |
| Flow Control ID | 16 | Flow identifier/hint available to forwarding and higher-layer logic. |
| Reserved | 5 | Must transmit as zero; ignored on receive. |
| Source Address | 58 | Hierarchical source GDP address. |
| Destination Address | 58 | Hierarchical destination GDP address. |

Flit 1 is intentionally exact: after Frame Type, `Type + Version + Size Class + Hop Limit + QoS` consumes all remaining 28 bits. Flit 2 consumes `8 + 16 + 5 = 29` carried bits. Each 58-bit address then consumes exactly two complete 29-bit continuation flits.

## Payload size classes

The four-bit Size Class gives sixteen fixed GDP payload budgets:

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

Class 1 is the smallest non-empty class and is intended to represent approximately one additional carried-data flit. The receiver determines the expected GDP payload size solely from Size Class.

## Header checksum

GDP contains an 8-bit header checksum for accidental header-corruption detection. It protects GDP routing/header information only, not the payload. The checksum algorithm is intentionally lightweight; the exact normative arithmetic definition should be kept consistent across all implementations.

Link-level corruption detection remains the responsibility of DLP. End-to-end payload integrity and reliability belong to GNet or another GDP-carried protocol.

## Processing rules

- A router validates the GDP header before forwarding.
- A router decrements Hop Limit on forwarding. A packet whose Hop Limit expires is dropped; a higher-level diagnostic/error protocol may report the condition.
- Reserved bits must be zero when transmitted and ignored by receivers for forward compatibility.
- Source and Destination addresses are transmitted most-significant bit first.
- GDP itself does not fragment packets. Path/link capability must constrain usable Size Classes where required.
