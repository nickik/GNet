---
id: glossary
title: "Glossary"
aliases: ["GNet glossary"]
type: meta
status: active
tags: ["gnet","gnet/meta","gnet/status/active"]
parent: "[[GNet Home]]"
related: ["[[Current Protocol Stack Summary]]","[[GNet Architecture Overview]]"]
updated: 2026-09-06
---
# Glossary

| Term | Meaning |
|---|---|
| Flit | The GNet flow-control/data unit: exactly 32 data bits. A flit does not include VCID bits. |
| Phit | Physical transfer unit of a particular PHY. One flit may occupy one or more phits; phit width and line encoding are PHY-specific. |
| VCID | Hop-local Virtual Channel Identifier associated with every flit; two bits in baseline VC2. |
| VC2 | Baseline link metadata width: a 2-bit VCID associated with a 32-bit flit. An inline serial representation carries 34 logical link bits per flit. |
| VC4 / VC8 | Possible future negotiated wider-VC options. They do not change the 32-bit flit data width and are not baseline profiles. |
| DLP | Direct Link Protocol, the minimal hop-local data-path contract. |
| GLCP | GNet Link Control Protocol; hop-local bootstrap, capability, credit, grant, VC, reset, and status control. |
| CREDIT | Guaranteed downstream receive capacity for one complete 32-bit flit and its associated link metadata, independent of PHY phit width. |
| GRANT | Infrastructure permission to consume some reserved credits now. |
| Scheduling quantum | Maximum scheduled transmission interval before infrastructure reconsiders access; not a credit unit. |
| GDP | GNet Datagram Protocol, the routed Layer-3 package protocol. |
| GDP Size Class | Four-bit GDP field selecting one of sixteen fixed payload budgets from empty through 1 MiB. |
| GDP address | 64-bit hierarchical routed address. |
| GCTL | GDP-carried network control for discovery/configuration/routing/OAM; distinct from GLCP. |
| GNet-3 | Universal native-copper baseline: 3 Mbit/s of flit data nominal with 1.5/0.75 fallback. |
| GNet-10 | Switched LAN profile providing 10 Mbit/s of flit data on independently negotiated ports. |
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
