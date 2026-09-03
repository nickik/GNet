---
id: glossary
title: "Glossary"
aliases: ["GNet glossary"]
type: meta
status: active
tags: ["gnet","gnet/meta","gnet/status/active"]
parent: "[[GNet Home]]"
related: ["[[Tag Index]]","[[GNet Architecture Overview]]"]
updated: 2026-09-02
---
# Glossary

> [!info] Knowledge graph
> **Up:** [[GNet Home]] · **Related:** [[Tag Index]] · [[GNet Architecture Overview]]


| Term | Meaning |
|---|---|
| DLP | Direct Link Protocol, the common L2 framing contract. |
| Flit | One complete 32-bit link transfer: a 4-bit VCID followed by 28 carried bits. |
| VCID | Four-bit, hop-local Virtual Channel Identifier repeated in every flit of one active bounded DLP segment. |
| DLP segment | A bounded sequence consisting of one VCID-tagged header, counted payload flits, and a link-integrity trailer. |
| Segment Class | Two-bit DLP field selecting the maximum meaningful payload: Small 64 octets, Normal 256 octets, or Large 1,024 octets. |
| Link Flow ID | Proposed persistent link-local identifier associating successive bounded segments with reservation or scheduling state; unlike a VCID, it need not be released after every segment. |
| GDP | GNet Datagram Protocol, the routed L3 protocol. |
| GNET-L | Local request/grant star link. |
| GNET-A | Centrally polled shared access network. |
| GNET-P | Dedicated synchronous point-to-point trunk. |
| GCTL | GNet control payload protocol. |
| GTS | GNet transport/session protocol family. |
| GTerm | Virtual terminal application protocol. |
| GSC | GNet Session Control, signaling for terminal, voice, and other interactive sessions. |
| Router domain | The address prefix and local links managed by one router service. |
| Directory | Named-service and identity-to-service lookup system. |
| Registrar | District or metro service that maintains current device reachability. |
| Flow | Traffic with a persistent identity and optional QoS reservation; distinct from the temporary hop-local VCID used to carry one segment. |
| Tunnel | Historical name for a GTS connection/session; retained in the draft packet fields. |
| Reset ID | Capability presented to authorize tunnel close, reset, or rebind. |
| Stream | Independently negotiated byte/message flow multiplexed inside a GTS tunnel. |
| DigitalKey | Removable identity/security card carrying identity and protected credentials. |
| QDX | Internal bus/architecture used by some systems; not a GNet protocol layer. |
