---
id: packet-formats-moc
title: "Packet Formats MOC"
aliases: ["Packet definitions"]
type: moc
status: active
tags: ["gnet","gnet/moc","gnet/status/active"]
parent: "[[GNet Home]]"
related: ["[[34-bit Flit Format]]","[[Virtual Channels and VCIDs]]","[[Protocols MOC]]"]
updated: 2026-09-03
---
# Packet formats map of content

Packet layouts use RFC-style 32-bit diagrams for logical words. A row labelled **Flit** is one actual transmitted flit and, for the baseline VC2 profile, shows a 2-bit VCID plus 32 carried bits. A row labelled **Word** is a logical protocol-layout aid.

- [[34-bit Flit Format]] — normative baseline flit and advanced VC-width concept.
- [[GLCP Control Flits]] — 34-bit link-bootstrap control flits for HELLO and CAPABILITIES.
- [[Virtual Channels and VCIDs]] — VC scope/lifecycle and advanced VC4 negotiation.
- [[GDP Datagram]] — minimal routed header and GDP Size Class registry.
- [[Discovery Packets]] — logical GCTL service discovery messages.
- [[Address Configuration Packets]] — logical GCTL address/delegation messages.
- [[GSC Packet]] — session-control envelope.
- [[GTS Transport Packets]] — higher-layer transport layouts.
- [[DLP Segment Size Classes]] — superseded historical DLP size-class model.

Future normative packets need field-validation rules, state-machine transitions, error behavior, and golden test vectors.
