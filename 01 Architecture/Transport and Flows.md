---
id: transport-and-flows
title: "Transport and Flows"
aliases: ["GNet transport architecture"]
type: architecture
status: mixed
layers: ["L4","L5"]
tags: ["gnet","gnet/architecture","gnet/status/mixed","gnet/layer/l4","gnet/layer/l5"]
parent: "[[Architecture MOC]]"
related: ["[[GTS Protocol]]","[[GTS Transport Packets]]","[[Canonical Service Selector]]","[[Virtual Channels and VCIDs]]","[[ADR-0005 Tunnels and Streams]]"]
updated: 2026-09-14
---
# Transport, sessions, and reserved flows

> [!info] Knowledge graph
> **Up:** [[Architecture MOC]] · **Related:** [[GTS Protocol]] · [[GTS Transport Packets]] · [[Canonical Service Selector]] · [[Virtual Channels and VCIDs]] · [[ADR-0005 Tunnels and Streams]]

Status: **FROZEN initial traditional packet-routed transport identity; selected timing constants remain open**

GDP provides routed datagrams and only header integrity. The initial GTS profile above it provides reliable ordered streams and mandatory end-to-end CRC protection.

## Tunnel and stream model

GTS first establishes a tunnel. The initial profile freezes:

- 32-bit receiver-local Tunnel IDs;
- 32-bit receiver-local Reset IDs;
- 8-bit Stream IDs scoped within one tunnel;
- service selection in CONNECT only;
- Stream 0 created by CONNECT;
- traditional packet-by-packet GDP routing with no flow-ID routing dependency.

Each endpoint allocates the Tunnel ID that the peer uses when sending to it. Ordinary GTS packets therefore carry the receiver's local Tunnel ID, not a globally unique connection identifier and not a source/destination pair of Tunnel IDs.

Stream IDs multiplex reliable ordered data streams within one tunnel. The CONNECT initiator owns even Stream IDs, the responder owns odd Stream IDs, and Stream 0 is created as part of CONNECT.

## Service binding

Service identity is separate from transport identity. GTS does not require TCP-style source/destination ports.

CONNECT carries exactly one [[Canonical Service Selector]] (CSS). If CONNECT succeeds, the resulting tunnel is bound to that canonical service identity for its lifetime.

CSS has one 128-bit canonical namespace with three wire representations:

- Registered-8: 8-bit registered compressed form such as `-FILE`;
- Short-32: four-character form such as `#CAPI`;
- Full-128: complete 128-bit CSS value.

Registered-8 and Short-32 expand into the same canonical CSS128 namespace. The shortest available representation is canonical.

Additional STREAM_OPEN operations do not contain a CSS and cannot select another service. They create more streams inside the already service-bound tunnel. Selecting another service requires a separate GTS tunnel.

Ordinary DATA therefore needs only:

```text
receiver-local Tunnel ID
Stream ID
Sequence
payload
CRC
```

and does not repeat service identity.

## Reliability baseline

The initial profile is reliable and ordered. Unreliable stream modes are deferred.

GTS uses packet-oriented 32-bit sequence numbers rather than byte sequence numbers. ACK uses a cumulative base plus a 32-bit selective bitmap and explicit receive credit. Retransmission timeout remains adaptive from measured RTT; exact estimator constants and bounds remain to be frozen separately.

## Integrity

Traditional GTS packets use mandatory CRC-32 end to end. The tiny CRC-8 rule is retained only for a future compact GTS representation capable of fitting the 0-byte/3-byte GDP classes. Ordinary 32-bit-Tunnel/8-bit-Stream GTS cannot fit those tiny classes.

GTS CRC binds the complete GTS header/payload to the canonical effective GDP source/destination endpoints and GDP Size Class. Local GDP address representations must expand to the same effective endpoint identities before CRC calculation.

## Deferred features

The initial traditional packet-routed profile does not require:

- flow-ID/virtual-circuit based routing optimization;
- multicast/group transport;
- unreliable streams;
- end-to-end congestion control;
- dynamic route exchange;
- cryptographic security.

These may be added later without changing the basic GDP packet-routed baseline.
