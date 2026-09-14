---
id: transport-and-flows
title: "Transport and Flows"
aliases: ["GNet transport architecture"]
type: architecture
status: mixed
layers: ["L4","L5"]
tags: ["gnet","gnet/architecture","gnet/status/mixed","gnet/layer/l4","gnet/layer/l5"]
parent: "[[Architecture MOC]]"
related: ["[[GTS Protocol]]","[[GTS Transport Packets]]","[[GTS Stream Profiles]]","[[Canonical Service Selector]]","[[Virtual Channels and VCIDs]]","[[ADR-0005 Tunnels and Streams]]"]
updated: 2026-09-14
---
# Transport, sessions, and reserved flows

Status: **FROZEN packet-routed transport identity and extended stream-profile model; selected timing constants remain open**

GDP provides routed datagrams and header integrity. GTS adds service-bound tunnels containing multiple streams with independently selected delivery, sizing, sequencing, payload-integrity and direction behavior.

## Tunnel and service model

The baseline freezes:

- 32-bit receiver-local Tunnel IDs;
- 32-bit receiver-local Reset IDs for tunnel RESET;
- 8-bit Stream IDs scoped within one tunnel;
- CONNECT selects one CSS and creates Stream 0;
- additional STREAM_OPEN operations remain inside that service-bound tunnel;
- CONNECT initiator owns even Stream IDs, responder owns odd Stream IDs.

Ordinary established traffic uses receiver-local Tunnel ID + Stream ID rather than repeating a service selector or source/destination port pair.

## Message-preserving transport

GTS is message-preserving.

One DATA, DATA_END, or DATAGRAM packet is one application message unit. Reliable streams preserve message order and message boundaries. Implementations may expose a byte-stream API by concatenating delivered messages, but that is an API convenience above the protocol.

## Stream Profile

Every stream carries a 16-bit profile:

```text
15      Unreliable
14      Variable
13      Sequenced
12      Unchecked Payload
11..10  Direction
9..4    Reserved
3..0    Size Class
```

Direction is relative to the stream opener:

```text
01   opener -> peer
10   peer -> opener
11   bidirectional
00   invalid/reserved
```

Stream 0 treats the CONNECT initiator as opener.

## Reliable streams

Reliable Fixed and Reliable Variable use packet/message sequence numbers, selective ACK, retransmission and GTS Receive Credit.

Reliable Variable allows each message packet to choose a GDP Size Class up to the negotiated maximum and therefore carries Valid Length.

Reliable streams always use full CRC-32 payload coverage. The explicit Sequenced bit is zero because sequencing is inherent.

## Unreliable streams

Unreliable streams use DATAGRAM and have no GTS ACK, retransmission or receive credit.

They may be unsequenced or sequenced. Sequenced unreliable datagrams carry a 32-bit sequence number solely for freshness/loss/duplicate handling; they are never retransmitted.

Fixed and Variable sizing are independent of sequencing.

## Unchecked payload

Unreliable streams may set Unchecked Payload. The CRC-32 field remains present and protects GDP pseudo-header plus all GTS transport metadata. Application payload and padding are excluded from the CRC.

This permits damaged media payload to be delivered while preventing corrupted Tunnel ID, Stream ID, Type, Sequence or Valid Length from being treated as valid.

Unchecked Payload is invalid on reliable streams.

## Unidirectional streams

Direction applies only to application data. Control packets needed to operate or retire a stream may still travel in either direction.

For reliable unidirectional streams, the non-receiving endpoint advertises zero GTS Receive Credit. All unreliable streams advertise zero receive credit in both directions.

## Stream reset

STREAM_RESET terminates one stream immediately in both directions without affecting the tunnel or sibling streams. STREAM_RESET_ACK confirms peer retirement of that stream.

This is distinct from tunnel RESET, which destroys the entire tunnel and requires the tunnel Reset ID.

## Deferred features

The baseline still does not require:

- flow-ID/virtual-circuit routing optimization;
- multicast/group transport;
- end-to-end congestion control;
- dynamic route exchange;
- cryptographic security.

See [[GTS Stream Profiles]] and [[GTS Transport Packets]] for normative details.
