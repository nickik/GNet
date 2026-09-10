---
id: gdp-protocol
title: "GDP Protocol"
aliases: ["GDP","Global Data Protocol"]
type: protocol
status: mixed
layers: ["L3"]
tags: ["gnet","gnet/protocol","gnet/status/mixed","gnet/layer/l3"]
parent: "[[Protocols MOC]]"
related: ["[[GDP Datagram]]","[[Addressing and Routing]]","[[ADR-0002 Minimal GDP Header]]","[[ADR-0015 Restore Minimal GDP Header]]"]
updated: 2026-09-10
---
# GNet Datagram Protocol (GDP)

Status: **FROZEN semantic field set; DRAFT encoding**

GDP is the common routed Layer-3 protocol. Its header remains deliberately minimal.

## Required semantics

- **Version** selects the GDP wire version.
- **Type** identifies the payload protocol.
- **Size Class** identifies the fixed GDP package payload size.
- **Hop Limit** is decremented at each GDP router; a packet reaching zero is discarded.
- **QoS** selects forwarding/service treatment subject to local policy.
- **Source** and **Destination** are 64-bit GDP addresses.

GDP MUST NOT acquire a payload-length field, checksum, CRC, hash, integrity flag/trailer, fragmentation state, options, flow/session ID, sequence number, acknowledgement, receive window, or encryption metadata.

DLP/GLCP supply hop-local transfer framing, integrity, credits, and scheduling. Endpoints supply end-to-end integrity, reliability, fragmentation/reassembly where required, and session state above GDP.

For GTS payloads, GTS supplies mandatory end-to-end CRC protection. The GTS CRC is bound to a GDP pseudo-header containing the relevant effective source endpoint, effective destination endpoint, GDP Type, and GDP Size Class. This binding is mandatory in both global GDP operation and any compact/local GDP encoding; local operation uses the corresponding effective local endpoint fields rather than omitting the GDP binding. Mutable forwarding fields such as Hop Limit are not part of the end-to-end pseudo-header.

The current GTS integrity rule uses CRC-8 only with the dedicated 0-byte and 3-byte tiny GDP classes and CRC-32 with every larger GDP Size Class. The CRC remains a GTS field/trailer and does not change GDP's no-integrity-field rule.

The current 20-octet wire-layout candidate is defined in [[GDP Datagram]]. The semantic rule is more important than the current bit packing: GDP is routing metadata plus package size, not a transport or link-flow protocol.
