---
id: adr-0002-minimal-gdp-header
title: "ADR-0002 Minimal GDP Header"
aliases: ["Decision 0002"]
type: decision
status: mixed
layers: ["L3"]
tags: ["gnet","gnet/decision","gnet/status/mixed","gnet/layer/l3"]
parent: "[[Decisions MOC]]"
related: ["[[GDP Protocol]]","[[GDP Datagram]]","[[ADR-0009 No GDP Integrity Field]]"]
updated: 2026-09-02
---
# Decision 0002: Minimal GDP header

> [!info] Knowledge graph
> **Up:** [[Decisions MOC]] · **Related:** [[GDP Protocol]] · [[GDP Datagram]] · [[ADR-0009 No GDP Integrity Field]]


Status: **FROZEN field set; DRAFT widths**

GDP contains exactly Version, Type, Hop Limit, QoS, Destination Address, and Source Address. Payload length is known from DLP. Integrity, fragmentation, options, reliability, flow/session identification, and encryption are intentionally excluded. The destination precedes the source in the current encoding candidate so forwarding can begin as soon as the destination is available.

The working encoding assigns one octet to each control field and eight octets to each address, producing exactly 20 logical octets. Because every transmitted flit reserves four bits for VCID, DLP carries this 160-bit header across six 28-bit carried regions rather than five transmitted flits. This allocation can change before version 1 without changing the architectural decision.

GDP contains no checksum, CRC, hash, or other integrity field. [[ADR-0009 No GDP Integrity Field]] records the integrity-layer decision explicitly.
