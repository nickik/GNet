---
id: gdp-datagram
title: "GDP Datagram"
aliases: ["GDP packet"]
type: packet
status: mixed
layers: ["L3"]
tags: ["gnet","gnet/packet","gnet/status/mixed","gnet/layer/l3"]
parent: "[[Packet Formats MOC]]"
related: ["[[GDP Protocol]]","[[32-bit Flit Format]]","[[ADR-0002 Minimal GDP Header]]"]
updated: 2026-09-02
---
# GDP datagram packet

> [!info] Knowledge graph
> **Up:** [[Packet Formats MOC]] · **Related:** [[GDP Protocol]] · [[32-bit Flit Format]] · [[ADR-0002 Minimal GDP Header]]


Status: **DRAFT five-flit encoding; FROZEN field set**

```text
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |    Version    |      Type     |   Hop Limit   |      QoS      | Flit 0
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                  Source Address [63:32]                       | Flit 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                  Source Address [31:0]                        | Flit 2
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                Destination Address [63:32]                    | Flit 3
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                Destination Address [31:0]                     | Flit 4
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                     GDP payload begins                        | Flit 5
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                              ...                              |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

The GDP header is exactly **five 32-bit flits (20 octets)**. Source and Destination are transmitted most-significant word first. The enclosing DLP Payload Length determines the GDP payload size; the DLP final-flit rule supplies and removes any zero padding.

A source of zero is permitted only during explicitly defined bootstrap exchanges. A normal router decrements Hop Limit before transmission; if the result is zero it discards the packet and may return a control error once such errors are defined.

Version 0 is reserved. The first interoperable version is expected to use Version 1. Type and QoS values are provisional registries. GDP itself has no padding field or optional header.
