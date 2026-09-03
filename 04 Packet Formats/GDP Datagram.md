---
id: gdp-datagram
title: "GDP Datagram"
aliases: ["GDP packet"]
type: packet
status: mixed
layers: ["L3"]
tags: ["gnet","gnet/packet","gnet/status/mixed","gnet/layer/l3"]
parent: "[[Packet Formats MOC]]"
related: ["[[GDP Protocol]]","[[32-bit Flit Format]]","[[Virtual Channels and VCIDs]]","[[ADR-0002 Minimal GDP Header]]","[[ADR-0009 No GDP Integrity Field]]"]
updated: 2026-09-02
---
# GDP datagram packet

> [!info] Knowledge graph
> **Up:** [[Packet Formats MOC]] · **Related:** [[GDP Protocol]] · [[32-bit Flit Format]] · [[Virtual Channels and VCIDs]] · [[ADR-0009 No GDP Integrity Field]]

Status: **DRAFT 20-octet encoding; FROZEN field set and absence of integrity field**

## Logical GDP header

GDP defines a continuous 160-bit header. The following diagram describes the protocol fields before DLP flit slicing; its rows are logical 32-bit words, not transmitted flits.

```text
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |    Version    |      Type     |   Hop Limit   |      QoS      |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                  Source Address [63:32]                       |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                  Source Address [31:0]                        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                Destination Address [63:32]                    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                Destination Address [31:0]                     |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                     GDP payload begins                        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

The logical GDP header is exactly 20 octets. It contains only Version, Type, Hop Limit, QoS, Source Address, and Destination Address.

## Actual DLP flit packing

Every transmitted flit is 32 bits: four VCID bits and 28 carried bits. The DLP segment header precedes the GDP bytes. The first six DLP payload flits containing GDP are therefore:

```text
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |    Version    |     Type      |   Hop Limit   | Q hi  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  | Q lo  |            Source Address [63:40]             |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |                Source Address [39:12]                 |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  | Source Address [11:0] |  Destination Address [63:48]  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |              Destination Address [47:20]              |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |      Destination Address [19:0]       | GDP Data[0:7] |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |                  GDP payload [8:35]                   |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |                          ...                          |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

`Q` denotes QoS. If GDP has at least one payload octet, its first eight bits occupy the final carried bits of Payload 5. If GDP has no payload, those eight positions and any other unused positions in the final DLP payload flit are DLP zero padding.

Source and Destination are transmitted most-significant bit first. The enclosing DLP Payload Length is `20 + M` octets for M GDP payload octets. DLP transmits `ceil((160 + 8M) / 28)` payload flits, followed by its own integrity trailer.

## Processing

A source of zero is permitted only during explicitly defined bootstrap exchanges. A router decrements Hop Limit before retransmission; if the result is zero it discards the packet and may return a control error once such errors are defined.

Version 0 is reserved. The first interoperable version is expected to use Version 1. Type and QoS values remain provisional registries.

> [!important] Integrity boundary
> GDP contains no header checksum, payload checksum, CRC, hash, integrity trailer, or integrity flag. Any DLP CRC belongs to the surrounding link segment and is removed at the receiving interface. End-to-end integrity belongs to GTS or the protocol carried by GDP.
