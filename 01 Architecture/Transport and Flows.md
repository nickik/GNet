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

Status: **FROZEN initial packet-routed transport identity and stream profiles; selected timing constants remain open**

GDP provides routed datagrams and header integrity. GTS adds service-bound tunnels containing multiple streams whose reliability and packet-size behavior are selected independently.

## Tunnel and stream model

The initial profile freezes:

- 32-bit receiver-local Tunnel IDs;
- 32-bit receiver-local Reset IDs;
- 8-bit Stream IDs scoped within one tunnel;
- service selection in CONNECT only;
- Stream 0 created by CONNECT;
- CONNECT initiator owns even Stream IDs, responder owns odd Stream IDs;
- traditional packet-by-packet GDP routing with no flow-ID routing dependency.

Each endpoint allocates the Tunnel ID that the peer uses when sending to it. Ordinary GTS packets carry only the receiver's local Tunnel ID rather than a globally unique connection identifier or source/destination Tunnel-ID pair.

## Service binding

CONNECT carries one [[Canonical Service Selector]] (CSS). If CONNECT succeeds, the tunnel is bound to that canonical service identity for its lifetime. Additional STREAM_OPEN operations create streams inside the same service-bound tunnel and do not carry CSS.

Ordinary established traffic therefore uses Tunnel ID + Stream ID rather than repeating a TCP-style source/destination service port pair.

## Stream profiles

Every stream has two independent profile bits:

```text
Unreliable = 0/1
Variable   = 0/1
```

producing four baseline profiles:

| Profile | Delivery | Packet size | GTS ACK/retransmit |
|---|---|---|---|
| Reliable Fixed | reliable ordered | one exact GDP Size Class | yes |
| Reliable Variable | reliable ordered | per-packet class up to a negotiated maximum | yes |
| Unreliable Fixed | independent datagrams | one exact GDP Size Class | no |
| Unreliable Variable | independent datagrams | per-packet class up to a negotiated maximum | no |

Stream 0 may use any of the four profiles. A single tunnel may mix profiles across its streams.

The common Stream Parameters byte is:

```text
bit  7      Unreliable
bit  6      Variable
bits 5..4   Reserved = 0
bits 3..0   Size Class
```

For Fixed streams, Size Class is exact. For Variable streams, it is the maximum allowed class.

## Reliable streams

Reliable streams use 32-bit packet-oriented sequence numbers, not byte sequence numbers. ACK uses a cumulative base, a 32-bit selective bitmap, and receive credit.

Reliable Variable DATA adds a 16-bit Valid Length because the chosen GDP Size Class may be larger than the application data carried in that packet. Packet numbering and ACK semantics remain unchanged regardless of packet size.

Receive Credit remains packet-based. On Reliable Variable streams, one credit guarantees capacity for one packet up to the negotiated maximum Size Class.

## Unreliable streams

Unreliable streams use GTS `DATAGRAM` packets. They have no GTS sequence number, acknowledgement, retransmission, duplicate suppression, ordering, or transport receive credit.

Unreliable Fixed DATAGRAM uses the stream's exact GDP Size Class. Unreliable Variable DATAGRAM adds Valid Length and may choose any class up to the stream maximum.

Applications that need media timestamps, sequence numbers, epochs, or loss detection carry those semantics themselves.

Lower-layer GNet hop-local credit/backpressure still applies; removing GTS Receive Credit does not allow one physical/link endpoint to overrun another.

## Integrity

The current baseline retains CRC-32-GNET for both reliable DATA and unreliable DATAGRAM packets. A CRC failure is discarded; reliable delivery can recover through retransmission, while an unreliable datagram is simply lost.

A possible future **header-only integrity profile for unreliable media** is still open. Completely unchecked GTS headers are not currently defined.

## Deferred features

The baseline still does not require:

- flow-ID/virtual-circuit routing optimization;
- multicast/group transport;
- end-to-end congestion control;
- dynamic route exchange;
- cryptographic security.

See [[GTS Stream Profiles]] and [[GTS Transport Packets]] for normative details.
