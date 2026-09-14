---
id: gts-protocol
title: "GTS Protocol"
aliases: ["GTS","GNet Transport and Session Protocol"]
type: protocol
status: frozen
layers: ["L4","L5"]
tags: ["gnet","gnet/protocol","gnet/status/frozen","gnet/layer/l4","gnet/layer/l5"]
parent: "[[Protocols MOC]]"
related: ["[[GTS Transport Packets]]","[[GTS Stream Profiles]]","[[Transport and Flows]]","[[Canonical Service Selector]]","[[ADR-0005 Tunnels and Streams]]","[[ADR-0018 GDP Header CRC and Local 16-bit Form]]"]
updated: 2026-09-14
---
# GNet Transport and Session Protocol (GTS)

Status: **FROZEN initial packet-routed protocol and four baseline stream profiles; selected timing constants and conformance vectors remain**

GTS is the endpoint transport/session protocol above GDP. One GTS tunnel is bound to one service selected by CSS and may contain multiple streams. Streams independently select reliable/unreliable delivery and fixed/variable packet sizing.

## Packet types

The GTS first byte contains Version in the high four bits and Type in the low four bits. Initial Version is `0`.

| Type | Name |
|---:|---|
| `0x0` | RESERVED |
| `0x1` | CONNECT |
| `0x2` | CONNECT_ACK |
| `0x3` | STREAM_OPEN |
| `0x4` | STREAM_ACK |
| `0x5` | DATA |
| `0x6` | ACK |
| `0x7` | DATA_END |
| `0x8` | STREAM_CLOSE |
| `0x9` | STREAM_CLOSE_ACK |
| `0xA` | TUNNEL_CLOSE |
| `0xB` | TUNNEL_CLOSE_ACK |
| `0xC` | RESET |
| `0xD` | DATAGRAM |
| `0xE`-`0xF` | RESERVED |

## Identifiers

- Tunnel ID: 32 bits, receiver-local.
- Reset ID: 32 bits, receiver-local capability used for RESET.
- Stream ID: 8 bits, scoped to one tunnel.
- Reliable Packet Sequence: 32 bits, stream-scoped and counts DATA/DATA_END packets rather than bytes.
- CONNECT initiator owns even Stream IDs; responder owns odd Stream IDs; Stream 0 is created by CONNECT.

Each endpoint allocates the Tunnel ID the peer uses when sending toward it. Ordinary packets therefore carry the receiver's Tunnel ID only.

## Stream profiles

The common Stream Parameters byte is:

```text
bit  7      Unreliable
bit  6      Variable
bits 5..4   Reserved = 0
bits 3..0   Size Class
```

This defines four baseline profiles:

```text
00  Reliable Fixed
01  Reliable Variable
10  Unreliable Fixed
11  Unreliable Variable
```

where the first conceptual bit is Unreliable and the second is Variable. The exact bit positions are those above.

For Fixed streams, Size Class is the exact GDP Size Class for every DATA/DATAGRAM packet. For Variable streams, Size Class is the maximum allowed class and each packet may select its own GDP Size Class up to that maximum.

See [[GTS Stream Profiles]].

## Reliable DATA

Reliable Fixed DATA:

```text
Version/Type          1 B
Tunnel ID             4 B
Stream ID             1 B
Sequence              4 B
Data                   N
CRC-32                4 B
```

Reliable Variable DATA adds:

```text
Valid Length          2 B
```

between Sequence and Data. Valid Length identifies meaningful application bytes; remaining bytes before CRC are zero padding.

DATA_END is used only by reliable streams and always includes Valid Length. It consumes a normal sequence number and indicates that no later DATA/DATA_END is generated in that sending direction.

## ACK and reliability

ACK applies only to reliable streams:

```text
Version/Type          1 B
Tunnel ID             4 B
Stream ID             1 B
ACK Base              4 B
Receive Bitmap        4 B
Receive Credit        1 B
Reserved              1 B
CRC-32                4 B
```

ACK Base cumulatively acknowledges every packet through that sequence. The 32-bit bitmap selectively reports the next 32 sequences. Receive Credit counts additional packets the receiver guarantees it can accept.

For Reliable Variable streams, one credit guarantees capacity for one packet up to the negotiated maximum Size Class.

Reliable streams use selective retransmission plus an adaptive retransmission timeout. Exact timer constants remain to be frozen.

## Unreliable DATAGRAM

Unreliable Fixed DATAGRAM:

```text
Version/Type          1 B
Tunnel ID             4 B
Stream ID             1 B
Data                   N
CRC-32                4 B
```

Unreliable Variable DATAGRAM adds:

```text
Valid Length          2 B
```

between Stream ID and Data.

Unreliable streams have no GTS sequence number, ACK, retransmission, ordering, duplicate suppression, loss detection, or GTS Receive Credit. Applications may carry their own media/application sequence and timestamp information when needed.

A receiver unable to accept an otherwise valid DATAGRAM may discard it. Normal lower-layer GNet hop-local credit/backpressure still applies.

## CONNECT and Stream 0

CONNECT selects exactly one CSS, creates the tunnel, and creates Stream 0 with any baseline stream profile:

```text
Version/Type              1 B
Initiator Receive Tunnel  4 B
Initiator Reset ID        4 B
Stream-0 Parameters       1 B
Initial Receive Credit    1 B
CSS Representation/Res.   1 B
CSS Selector              1/4/16 B
Padding                   P
CRC-32                    4 B
```

For unreliable Stream 0, Initial Receive Credit MUST be zero.

The CSS representation field selects Registered-8, Short-32, or Full-128 as defined by [[Canonical Service Selector]]. A successful CONNECT binds the canonical CSS to the tunnel for its lifetime.

CONNECT_ACK returns responder Tunnel ID, responder Reset ID, status, and initial reliable receive credit. For unreliable Stream 0 its Initial Receive Credit MUST be zero.

CONNECT_ACK status `6` means unsupported Stream-0 profile.

## Additional streams

STREAM_OPEN contains:

```text
Version/Type           1 B
Tunnel ID              4 B
Stream ID              1 B
Stream Parameters      1 B
Initial Receive Credit 1 B
Reserved               1 B
Padding                P
CRC-32                 4 B
```

For unreliable streams, Initial Receive Credit MUST be zero.

STREAM_ACK accepts/rejects the proposed profile and returns initial credit for reliable streams. Status `6` means unsupported stream profile.

STREAM_OPEN does not carry CSS and cannot change service identity. A different service requires a separate CONNECT/tunnel.

## Stream close

STREAM_CLOSE remains directional.

For reliable streams, Final Sequence identifies the final reliable packet and STREAM_CLOSE_ACK confirms receipt through that sequence.

For unreliable streams, Final Sequence MUST be zero. STREAM_CLOSE/STREAM_CLOSE_ACK only synchronize retirement of stream state and make no claim that earlier DATAGRAM packets were delivered.

## Tunnel close and RESET

After all streams are retired, TUNNEL_CLOSE/TUNNEL_CLOSE_ACK gracefully retire the tunnel.

RESET immediately destroys a tunnel and every stream. It carries the receiver-local Reset ID established during CONNECT/CONNECT_ACK and is not acknowledged.

Retired Tunnel IDs and Stream IDs are protected by stale-state guard time before reuse.

## End-to-end integrity

GDP protects only its own immutable header information. The baseline GTS profile retains CRC-32-GNET over the canonical GDP pseudo-header plus the complete GTS header/content/padding for both reliable DATA and unreliable DATAGRAM packets.

```text
CRC-32-GNET
polynomial  0x04C11DB7
init        0xFFFFFFFF
refin       false
refout      false
xorout      0xFFFFFFFF
check       "123456789" -> 0xFC891918
```

A CRC failure is discarded. Reliable streams can recover through retransmission; unreliable streams treat it as packet loss.

Alternative integrity coverage for unreliable media, such as header-only CRC, remains open and is not part of the frozen baseline.

## Service selection

Service selection is CONNECT-only and uses [[Canonical Service Selector]]. The GDP address identifies the endpoint; CSS identifies the service. Ordinary DATA/DATAGRAM carries neither CSS nor TCP-style port numbers.

## Explicitly deferred

- flow-ID/virtual-circuit routing optimization;
- multicast/group transport;
- end-to-end congestion control;
- cryptographic authentication/encryption;
- dynamic routing behavior.

## Remaining work

- exact delayed-ACK/RTO constants for reliable streams;
- detailed malformed-control error mapping;
- golden packet/CRC/conformance vectors;
- decide whether unreliable media needs a header-only integrity profile.
