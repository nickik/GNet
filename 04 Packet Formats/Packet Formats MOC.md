---
id: packet-formats-moc
title: "Packet Formats MOC"
aliases: ["Packet definitions"]
type: moc
status: active
tags: ["gnet","gnet/moc","gnet/status/active"]
parent: "[[GNet Home]]"
related: ["[[32-bit Flit Format]]","[[Virtual Channels and VCIDs]]","[[Protocols MOC]]"]
updated: 2026-09-02
---
# Packet formats map of content

> [!info] Knowledge graph
> **Up:** [[GNet Home]] · **Related:** [[32-bit Flit Format]] · [[Virtual Channels and VCIDs]] · [[Protocols MOC]]


Packet layouts use RFC-style 32-bit diagrams. A row labelled **Flit** is one actual transmitted flit and must show its 4-bit VCID plus 28 carried bits. A row labelled **Word** is only a logical protocol-layout aid and is never a transmitted flit. Multi-octet fields use network byte order and are packed continuously across 28-bit carried regions.

- [[32-bit Flit Format]] — normative 4+28 format, bit order, padding, integrity trailer, and encapsulation.
- [[Virtual Channels and VCIDs]] — VCID scope, multiplexing, bounded reuse, and required state.
- [[DLP Segment Size Classes]] — 64-, 256-, and 1,024-octet segment limits and flit counts.
- [[GDP Datagram]] — fixed 20-octet header and actual six-flit DLP boundary.
- [[Discovery Packets]] — SOLICIT and ADVERTISE.
- [[Address Configuration Packets]] — ADDRESS_OFFER, ADDRESS_CLAIM, ACK, and NAK.
- [[GSC Packet]] — common session-control envelope.
- [[GTS Transport Packets]] — tunnel/stream requirements and the rejected historical packing.

Every future normative packet needs field validation rules, state-machine transitions, error behavior, and hexadecimal test vectors.
