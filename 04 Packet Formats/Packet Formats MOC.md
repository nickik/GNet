---
id: packet-formats-moc
title: "Packet Formats MOC"
aliases: ["Packet definitions"]
type: moc
status: active
tags: ["gnet","gnet/moc","gnet/status/active"]
parent: "[[GNet Home]]"
related: ["[[32-bit Flit Format]]","[[Virtual Channels and VCIDs]]","[[Protocols MOC]]"]
updated: 2026-09-06
---
# Packet formats map of content

Packet layouts use RFC-style 32-bit diagrams.

- A row labelled **Flit** is one actual 32-bit DLP data flit.
- A row labelled **Word** is a logical 32-bit protocol-layout unit.
- Baseline VCID metadata is associated with a flit but is not drawn inside its 32-bit data row.
- A specific PHY document may separately show VC metadata, framing, and physical transfer units (**phits**).

Because the architectural flit data width is 32 bits, a word-aligned 32-bit protocol word may occupy one flit directly. Protocol-specific unaligned fields may still span words/flits.

- [[32-bit Flit Format]] — normative 32-bit data-flit and PHY-phit model.
- [[Virtual Channels and VCIDs]] — baseline VC2 scope/lifecycle and future wider-VC possibilities.
- [[GDP Datagram]] — minimal routed header and GDP Size Class registry.
- [[Discovery Packets]] — logical GCTL service discovery messages.
- [[Address Configuration Packets]] — logical GCTL address/delegation messages.
- [[GSC Packet]] — session-control envelope.
- [[GTS Transport Packets]] — higher-layer transport layouts.
- [[DLP Segment Size Classes]] — superseded historical DLP size-class model.

Future normative packets need field-validation rules, state-machine transitions, error behavior, and golden test vectors.
