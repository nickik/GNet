---
id: glossary
title: "Glossary"
aliases: ["GNet glossary"]
type: meta
status: active
tags: ["gnet","gnet/meta","gnet/status/active"]
parent: "[[GNet Home]]"
related: ["[[Current Protocol Stack Summary]]","[[GNet Architecture Overview]]"]
updated: 2026-09-03
---
# Glossary

| Term | Meaning |
|---|---|
| Flit | One complete baseline 32-bit physical transfer: 2-bit VCID and 30 carried bits. |
| VCID | Hop-local Virtual Channel Identifier; two bits in the baseline VC2 profile. |
| VC2 | Baseline `2-bit VCID + 30 carried bits` flit profile. |
| VC4 | Future negotiated `4-bit VCID + 28 carried bits` advanced profile. |
| DLP | Direct Link Protocol, the minimal hop-local data-path contract. |
| GLCP | GNet Link Control Protocol; hop-local bootstrap, capability, credit, grant, VC, reset, and status control. |
| CREDIT | Guaranteed downstream receive capacity for one physical flit. |
| GRANT | Infrastructure permission to consume some reserved credits now. |
| Scheduling quantum | Maximum scheduled transmission interval before infrastructure reconsiders access; not a credit unit. |
| GDP | GNet Datagram Protocol, the routed Layer-3 package protocol. |
| GDP Size Class | Four-bit GDP field selecting one of sixteen fixed payload budgets from empty through 1 MiB. |
| GDP address | 64-bit hierarchical routed address. |
| GCTL | GDP-carried network control for discovery/configuration/routing/OAM; distinct from GLCP. |
| GNet-3 | Universal native-copper baseline: 3 Mbit/s nominal with 1.5/0.75 fallback. |
| GNet-10 | Switched LAN profile using independently negotiated 10 Mbit/s ports. |
| GNet-20 | Future bonded-lane copper profile with in-band control; not yet frozen. |
| GC | GNet Coupler: centrally arbitrated shared-medium LAN infrastructure. |
| GS | GNet Switch: active destination-specific multi-path LAN infrastructure. |
| GMC-8 | GNet Modular Connector, 8-contact; four balanced pairs. |
| GNET-A | Centrally scheduled residential-access profile. |
| GNET-P | Dedicated point-to-point infrastructure/trunk profile. |
| GTS | Higher-layer transport/tunnel/stream protocol carried by GDP. |
| Router domain | Address prefix and local links managed/advertised by an authorized routing service. |
| Directory | Named-service and identity-to-service lookup system. |
| DigitalKey | Removable identity/security card carrying identity and protected credentials. |
| QDX | Internal queued-device programming model used by implementations; not a GNet network layer. |
