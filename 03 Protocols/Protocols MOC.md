---
id: protocols-moc
title: "Protocols MOC"
aliases: ["Protocol Map of Content"]
type: moc
status: active
tags: ["gnet", "gnet/moc", "gnet/protocol", "gnet/status/active"]
parent: "[[GNet Home]]"
related: ["[[Architecture MOC]]", "[[Packet Formats MOC]]", "[[Registries MOC]]"]
updated: 2026-09-14
---
# Protocols map of content

> [!info] Knowledge graph
> **Up:** [[GNet Home]] · **Related:** [[Architecture MOC]] · [[Packet Formats MOC]] · [[Registries MOC]]

## Network and control

- [[Virtual Channels and VCIDs]] — hop-local active-segment multiplexing below GDP.
- [[GDP Protocol]] — routed global datagram.
- [[GCTL Protocol]] — network control.

## Transport and sessions

- [[GTS Protocol]] — tunnels, reset authority, streams and delivery.
- [[Canonical Service Selector]] — one 128-bit service namespace with Registered-8, Short-32, and Full-128 representations.
- [[GSC Protocol]] — GNet Session / SIP-like interactive-session signaling.
- [[Voice over GNet]] — native packet-voice profile using GSC signaling plus admitted realtime media flows.
- [[GTerm Protocol]] — routable virtual terminals.

Follow packet encodings through [[Packet Formats MOC]] and numeric assignments through [[Registries MOC]].
