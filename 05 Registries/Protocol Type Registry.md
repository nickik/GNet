---
id: protocol-type-registry
title: "Protocol Type Registry"
aliases: ["DLP and GDP protocol types"]
type: registry
status: draft
tags: ["gnet","gnet/registry","gnet/status/draft"]
parent: "[[Registries MOC]]"
related: ["[[Direct Link Protocol]]","[[GDP Protocol]]"]
updated: 2026-09-02
---
# Protocol type registry

> [!info] Knowledge graph
> **Up:** [[Registries MOC]] · **Related:** [[Direct Link Protocol]] · [[GDP Protocol]]


Status: **DRAFT allocations**

## DLP Protocol

| Value | Name | Meaning |
|---:|---|---|
| 0x00 | Reserved | invalid/unassigned |
| 0x01 | GDP | routed GNet datagram |
| 0x02 | GCTL-LINK | link-local GNet control |
| 0x10 | DECnet | native DECnet frame/packet encapsulation |
| 0x11 | IPv4 | Internet Protocol version 4 |
| 0x12 | XNS | Xerox Network Systems |
| 0x13 | SNA-GW | SNA gateway payload |
| 0xFF | Experimental | local experiments; never routed by default |

## GDP Type

| Value | Name | Meaning |
|---:|---|---|
| 0x00 | Reserved | invalid/unassigned |
| 0x01 | GCTL | routed control |
| 0x02 | GTS | GNet transport/session |
| 0x03 | GDM | unreliable application datagram |
| 0x10 | GTerm | virtual terminal |
| 0x11 | Boot | network boot service |
| 0x12 | Directory | names and service lookup |
| 0x13 | RPC | remote procedure call |
| 0x14 | Voice | packet voice payload/control demultiplexing |
| 0x15 | GSC | interactive-session signaling |
| 0xFF | Experimental | controlled experiments |

Registry values are placeholders until a compatibility policy is accepted.
