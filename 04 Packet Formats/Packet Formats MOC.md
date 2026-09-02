---
id: packet-formats-moc
title: "Packet Formats MOC"
aliases: ["Packet definitions"]
type: moc
status: active
tags: ["gnet","gnet/moc","gnet/status/active"]
parent: "[[GNet Home]]"
related: ["[[32-bit Flit Format]]","[[Protocols MOC]]"]
updated: 2026-09-02
---
# Packet formats map of content

> [!info] Knowledge graph
> **Up:** [[GNet Home]] · **Related:** [[32-bit Flit Format]] · [[Protocols MOC]]


Packet layouts use RFC-style 32-bit diagrams. Every diagram row is one transmitted flit; multi-octet integers use network byte order.

- [[32-bit Flit Format]] — notation, word order, padding, CRC trailer, and encapsulation.
- [[GDP Datagram]] — fixed five-flit GDP header.
- [[Discovery Packets]] — SOLICIT and ADVERTISE.
- [[Address Configuration Packets]] — ADDRESS_OFFER, ADDRESS_CLAIM, ACK, and NAK.
- [[GSC Packet]] — common session-control envelope.
- [[GTS Transport Packets]] — tunnel/stream requirements and the rejected historical packing.

Every future normative packet needs field validation rules, state-machine transitions, error behavior, and hexadecimal test vectors.
