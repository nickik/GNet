---
id: transport-and-flows
title: "Transport and Flows"
aliases: ["GNet transport architecture"]
type: architecture
status: mixed
layers: ["L4","L5"]
tags: ["gnet","gnet/architecture","gnet/status/mixed","gnet/layer/l4","gnet/layer/l5"]
parent: "[[Architecture MOC]]"
related: ["[[GTS Protocol]]","[[GTS Transport Packets]]","[[Virtual Channels and VCIDs]]","[[ADR-0005 Tunnels and Streams]]"]
updated: 2026-09-09
---
# Transport, sessions, and reserved flows

> [!info] Knowledge graph
> **Up:** [[Architecture MOC]] · **Related:** [[GTS Protocol]] · [[GTS Transport Packets]] · [[Virtual Channels and VCIDs]] · [[ADR-0005 Tunnels and Streams]]


Status: **FROZEN placement; OPEN exact wire protocol**

GDP provides best-effort routed datagrams and no integrity field. Endpoints add the following functions as needed:

- setup-only service selection using variable-width Service Selectors;
- session/tunnel identity and local handles;
- stream multiplexing within a tunnel;
- sequence and acknowledgment numbers;
- receive windows and flow control;
- retransmission and ordered delivery;
- fragmentation and reassembly when payloads exceed a link path limit;
- end-to-end integrity and optional encryption;
- reserved-flow setup and release.

## Tunnel and stream model

GTS first establishes a tunnel. A separate Reset ID is the capability required to close, reset, or rebind it; normal data does not repeat that capability. Multiple streams share the tunnel and negotiate reliability, ordering/sequencing, byte-versus-message delivery, encryption, and compression independently.

Service identity is separate from transport identity. GTS does not require TCP-style source/destination ports. A setup-only Service Selector identifies the logical service during CONNECT or STREAM_OPEN; Tunnel IDs identify established associations and Stream IDs identify flows within them. Ordinary DATA packets do not repeat the Service Selector.

The accepted Service Selector model uses a 2-bit size class:

- class 0: 8-bit numeric registered service code;
- class 1: 32-bit fixed-width ASCII service name;
- class 2: 128-bit fixed-width ASCII service name;
- class 3: reserved.

The class-0 form requires only 10 logical bits and is intended for common services and very small systems. The 128-bit textual form permits sparse private or opaque service names, making exhaustive scanning impractical when names are sufficiently unpredictable. This is not cryptographic protection: predictable names remain guessable and passive observers can learn selectors from setup traffic.

The current working proposal uses a 64-bit Tunnel ID, 64-bit Reset ID, 16-bit Stream IDs, stream 0 for control, and stream 1 as default data. Only the tunnel-first model, reset-authority behavior, and variable-width service-selector semantics are accepted; these identifier widths and reserved stream numbers remain DRAFT.

The architecture should support at least three service modes: unreliable datagram, reliable ordered stream/message, and reserved real-time flow. They may share a common session-control header, but routers must not need transport state for ordinary forwarding.

## Real-time flows

Voice, interactive media, and other bounded-delay traffic remain GDP packets. Endpoints request resources through GNet Session Control. Routers perform admission control and schedule the admitted traffic using GDP QoS plus local reservation state. Legacy telephone circuits terminate only at district or edge gateways.

The persistent flow or reservation identity is distinct from the four-bit VCID. A router may bind successive bounded DLP segments of one reserved flow to temporary outgoing VCIDs. The VCID is released after each segment and never becomes an end-to-end transport identifier.

## Historical transport draft

The project previously selected detailed CONNECT and CONNECT_ACK field lists, including fixed source/destination port concepts. Their totals are not octet-aligned and CONNECT contains two receive-window fields. They are preserved in project history as requirements evidence, but are superseded for service selection by the variable-width Service Selector model and are not yet an interoperable encoding.
