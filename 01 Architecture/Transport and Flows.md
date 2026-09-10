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
updated: 2026-09-10
---
# Transport, sessions, and reserved flows

> [!info] Knowledge graph
> **Up:** [[Architecture MOC]] · **Related:** [[GTS Protocol]] · [[GTS Transport Packets]] · [[Virtual Channels and VCIDs]] · [[ADR-0005 Tunnels and Streams]]

Status: **FROZEN initial traditional packet-routed transport identity; OPEN exact packet layouts**

GDP provides best-effort routed datagrams and no end-to-end integrity field. The initial GTS profile above it provides reliable ordered streams and mandatory end-to-end CRC protection.

## Tunnel and stream model

GTS first establishes a tunnel. The initial profile freezes:

- 32-bit receiver-local Tunnel IDs;
- 8-bit Stream IDs scoped within one tunnel;
- service selection only during setup, not in ordinary DATA;
- traditional packet-by-packet GDP routing with no flow-ID routing dependency.

Each endpoint allocates the Tunnel ID that the peer uses when sending to it. Ordinary GTS packets therefore carry the receiver's local Tunnel ID, not a globally unique connection identifier and not a source/destination pair of Tunnel IDs.

Stream IDs multiplex reliable ordered data streams within a tunnel. Reserved Stream IDs and simultaneous-open allocation rules remain to be frozen with the exact STREAM_OPEN state machine.

Service identity is separate from transport identity. GTS does not require TCP-style source/destination ports. A setup-only Service Selector identifies the logical service during CONNECT or STREAM_OPEN; ordinary DATA packets use Tunnel ID and Stream ID.

The accepted Service Selector model uses a 2-bit size class:

- class 0: 8-bit numeric registered service code;
- class 1: short fixed-width textual selector;
- class 2: 128-bit fixed-width textual/private selector;
- class 3: reserved.

The exact compact character encoding for textual selectors remains open.

## Reliability baseline

The initial profile is reliable and ordered. Unreliable stream modes are deferred. Packet sequence/ACK representation, receive credit, retransmission timing, and the exact DATA/ACK layouts remain open.

The current direction is packet-oriented sequencing rather than TCP-style byte sequence numbers. A fixed ACK bitmap/selective-repeat scheme is under consideration but is not yet frozen.

## Integrity

Traditional GTS packets use mandatory CRC-32 end to end. The tiny CRC-8 rule is retained only for a future compact GTS representation capable of fitting the 0-byte/3-byte GDP classes. Ordinary 32-bit-Tunnel/8-bit-Stream GTS cannot fit those tiny classes.

GTS CRC binds the complete GTS header/payload to the canonical effective GDP source/destination endpoints and GDP Size Class. Local/compact GDP address representations must expand to the same effective endpoint identities before CRC calculation.

## Deferred features

The initial traditional packet-routed profile does not require:

- flow-ID/virtual-circuit based routing optimization;
- multicast/group transport;
- unreliable streams;
- end-to-end congestion control;
- dynamic route exchange;
- cryptographic security.

These may be added later without changing the basic GDP packet-routed baseline.
