---
id: gdp-protocol
title: "GDP Protocol"
aliases: ["GDP","Global Data Protocol"]
type: protocol
status: mixed
layers: ["L3"]
tags: ["gnet","gnet/protocol","gnet/status/mixed","gnet/layer/l3"]
parent: "[[Protocols MOC]]"
related: ["[[GDP Datagram]]","[[Addressing and Routing]]","[[ADR-0002 Minimal GDP Header]]"]
updated: 2026-09-02
---
# GNet Datagram Protocol (GDP)

> [!info] Knowledge graph
> **Up:** [[Protocols MOC]] · **Related:** [[GDP Datagram]] · [[Addressing and Routing]] · [[ADR-0002 Minimal GDP Header]]


Status: **FROZEN field set; DRAFT encoding**

GDP is the common routed L3 protocol. Its header is fixed and deliberately contains only six fields.

## Required semantics

- **Version** selects the GDP wire version.
- **Type** identifies the payload protocol.
- **Hop Limit** is decremented at each GDP router; a packet reaching zero is discarded.
- **QoS** selects forwarding and scheduling behavior, subject to local policy.
- **Source** and **Destination** are 64-bit GDP addresses.

GDP MUST NOT acquire length, checksum/hash, fragmentation, options, flow/session ID, sequence numbers, acknowledgments, or encryption metadata. DLP supplies frame length and link CRC; endpoints supply the rest above GDP.

The current encoding is exactly five 32-bit flits (20 octets) and is defined in [[GDP Datagram]]. Protocol Type and QoS allocations are DRAFT.
