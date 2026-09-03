---
id: glossary
title: "Glossary"
aliases: ["GNet glossary"]
type: meta
status: active
tags: ["gnet","gnet/meta","gnet/status/active"]
parent: "[[GNet Home]]"
related: ["[[Tag Index]]","[[Current Protocol Stack Summary]]","[[GNet Architecture Overview]]"]
updated: 2026-09-03
---
# Glossary

> [!info] Knowledge graph
> **Up:** [[GNet Home]] · **Related:** [[Current Protocol Stack Summary]] · [[GNet Architecture Overview]]

| Term | Meaning |
|---|---|
| DLP | Direct Link Protocol, the minimal Layer-2 direct-link framing contract. |
| Flit | One complete 32-bit link transfer: 2-bit VCID, 1-bit SOF, and 29 carried bits. |
| SOF | Start-of-frame bit. Set on the first flit of a frame. |
| Frame Type | One-bit discriminator in the first carried bit of the first flit. Currently GDP Data or Hello. |
| VCID | Two-bit, hop-local Virtual Channel Identifier repeated on flits of an active transfer. |
| Hello | Single-flit, link-local presence frame; never routed. |
| GDP | Global Data Protocol, the routed Layer-3 datagram protocol. |
| GDP Size Class | Four-bit GDP field selecting one of sixteen fixed payload sizes from empty through 1 MiB jumbogram. |
| GDP Flow Control ID | Sixteen-bit GDP header field used as a flow identifier/hint for forwarding and higher-layer flow control. |
| GDP address | 58-bit hierarchical routed address. |
| GNet | Layer-4/5 connection, tunnel, reliability, and stream protocol carried by GDP. |
| Tunnel ID | 64-bit GNet tunnel identity carried above GDP. |
| Router domain | The address prefix and local links managed by one router service. |
| Directory | Named-service and identity-to-service lookup system. |
| Registrar | District or metro service that maintains current device reachability. |
| Flow | Traffic with persistent higher-layer identity and optional QoS state; distinct from the temporary hop-local VCID. |
| GNET-L | Local request/grant star link profile. |
| GNET-A | Centrally polled shared-access profile. |
| GNET-P | Dedicated synchronous point-to-point trunk profile. |
| GTerm | Virtual terminal application protocol. |
| DigitalKey | Removable identity/security card carrying identity and protected credentials. |
| QDX | Internal bus/architecture used by some systems; not a GNet protocol layer. |
