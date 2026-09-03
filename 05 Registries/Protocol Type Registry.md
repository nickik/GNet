---
id: protocol-type-registry
title: "Protocol Type Registry"
aliases: ["DLP and GDP protocol types"]
type: registry
status: draft
tags: ["gnet","gnet/registry","gnet/status/draft"]
parent: "[[Registries MOC]]"
related: ["[[Direct Link Protocol]]","[[GDP Protocol]]","[[GCTL Protocol]]"]
updated: 2026-09-03
---
# Protocol type registry

Status: **DRAFT allocations**

## Native DLP carriage

Native GNet data VCs normally carry GDP. The former `GCTL-LINK` direct-DLP control type is superseded: GLCP owns native hop-local link control and GCTL is GDP-carried network control.

Non-GDP DLP adaptation profiles (for legacy protocols such as DECnet/IP/XNS) MAY define an explicit protocol selector in their VC/allocation metadata. Exact numeric DLP adaptation values remain OPEN and are not frozen by this table.

## GDP Type candidate registry

| Value | Name | Meaning |
|---:|---|---|
| 0x00 | Reserved | invalid/unassigned |
| 0x01 | GCTL | routed/bootstrap network control |
| 0x02 | GTS | GNet transport/session |
| 0x03 | GDM | unreliable application datagram |
| 0x10 | GTerm | virtual terminal |
| 0x11 | Boot | network boot service |
| 0x12 | Directory | names and service lookup |
| 0x13 | RPC | remote procedure call |
| 0x14 | Voice | packet voice payload/control demultiplexing |
| 0x15 | GSC | interactive-session signaling |
| 0xFF | Experimental | controlled experiments |

Allocations remain placeholders until the compatibility/versioning policy is accepted.
