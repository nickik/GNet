---
id: adr-0002-minimal-gdp-header
title: "ADR-0002 Minimal GDP Header"
aliases: ["Decision 0002"]
type: decision
status: mixed
layers: ["L3"]
tags: ["gnet","gnet/decision","gnet/status/mixed","gnet/layer/l3"]
parent: "[[Decisions MOC]]"
related: ["[[GDP Protocol]]","[[GDP Datagram]]","[[ADR-0009 No GDP Integrity Field]]","[[ADR-0017 32-bit Data Flit and PHY Phits]]"]
updated: 2026-09-06
---
# Decision 0002: Minimal GDP header

> [!info] Knowledge graph
> **Up:** [[Decisions MOC]] · **Related:** [[GDP Protocol]] · [[GDP Datagram]] · [[ADR-0009 No GDP Integrity Field]]

Status: **FROZEN field set; DRAFT widths**

GDP contains exactly Version, Type, Hop Limit, QoS, Source Address, and Destination Address. Payload length is known from DLP. Integrity, fragmentation, options, reliability, flow/session identification, and encryption are intentionally excluded.

The current working encoding adds the GDP Size Class within the compact first word and produces exactly **20 logical octets / 160 bits**. Under [[ADR-0017 32-bit Data Flit and PHY Phits]], this occupies exactly **five 32-bit data flits**. Hop-local VC metadata is associated by the link layer and consumes none of those 160 GDP bits.

The exact control-field widths remain subject to the current GDP encoding decision without changing the architectural requirement for a minimal header.

GDP contains no checksum, CRC, hash, or other integrity field. The restored current integrity-layer decision is indexed by [[ADR-0015 Restore Minimal GDP Header]].
