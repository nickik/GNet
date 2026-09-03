---
id: gdp-protocol
title: "GDP Protocol"
aliases: ["GDP","Global Data Protocol"]
type: protocol
status: mixed
layers: ["L3"]
tags: ["gnet","gnet/protocol","gnet/status/mixed","gnet/layer/l3"]
parent: "[[Protocols MOC]]"
related: ["[[GDP Datagram]]","[[Addressing and Routing]]","[[ADR-0002 Minimal GDP Header]]","[[ADR-0009 No GDP Integrity Field]]"]
updated: 2026-09-02
---
# GNet Datagram Protocol (GDP)

> [!info] Knowledge graph
> **Up:** [[Protocols MOC]] · **Related:** [[GDP Datagram]] · [[Addressing and Routing]] · [[ADR-0002 Minimal GDP Header]] · [[ADR-0009 No GDP Integrity Field]]


Status: **FROZEN field set; DRAFT encoding**

GDP is the common routed L3 protocol. Its header is fixed and deliberately contains only six fields.

## Required semantics

- **Version** selects the GDP wire version.
- **Type** identifies the payload protocol.
- **Hop Limit** is decremented at each GDP router; a packet reaching zero is discarded.
- **QoS** selects forwarding and scheduling behavior, subject to local policy.
- **Source** and **Destination** are 64-bit GDP addresses.

GDP MUST NOT acquire length, checksum, CRC, hash, integrity flag, integrity trailer, fragmentation, options, flow/session ID, sequence numbers, acknowledgments, or encryption metadata. DLP supplies segment length and hop-local integrity; endpoints supply end-to-end integrity and the remaining functions above GDP.

The current encoding is exactly 20 octets and is defined in [[GDP Datagram]]. It is not five transmitted flits: DLP carries the 160 header bits across six 28-bit carried regions, each prefixed by a 4-bit VCID. The sixth region may also contain the first eight payload bits. Protocol Type and QoS allocations are DRAFT.
