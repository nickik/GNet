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

Status: **FROZEN packet-routed transport/session architecture and extended stream-profile model; selected timing constants and conformance vectors remain**

GTS is the endpoint transport/session protocol above GDP. One GTS tunnel is bound to one service selected by CSS and may contain multiple independent streams. GTS is message-preserving: each DATA, DATA_END, or DATAGRAM packet represents one application message unit.

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
| `0xE` | STREAM_RESET |
| `0xF` | STREAM_RESET_ACK |

Unknown types are discarded.

## Identifiers

- Tunnel ID: 32 bits, receiver-local.
- Reset ID: 32 bits, receiver-local capability used for tunnel RESET.
- Stream ID: 8 bits, scoped to one tunnel.
- Packet Sequence: 32 bits where a stream profile requires sequencing.
- CONNECT initiator owns even Stream IDs; responder owns odd Stream IDs; Stream 0 is created by CONNECT.

Each endpoint allocates the Tunnel ID the peer uses when sending toward it. Ordinary established packets carry only the receiver's Tunnel ID.

## Message preservation

GTS preserves message boundaries.

Each DATA, DATA_END, or DATAGRAM packet is delivered as one application message unit. Reliable streams preserve both message ordering and message boundaries. GTS does not merge adjacent messages into a byte stream and does not split one message across multiple GTS data packets in the baseline profile.

An operating system or library MAY present a conventional byte-stream API by concatenating delivered reliable messages. That is an API adaptation above GTS and does not alter wire semantics.

## Stream Profile

CONNECT and STREAM_OPEN carry a 16-bit Stream Profile:

```text
bit 15      Unreliable
bit 14      Variable
bit 13      Sequenced
bit 12      Unchecked Payload
bits 11..10 Direction
bits 9..4   Reserved = 0
bits 3..0   Size Class
```

Direction is relative to the endpoint that opened the stream:

```text
00   invalid / reserved
01   opener -> peer only
10   peer -> opener only
11   bidirectional
```

For Stream 0, the opener is the CONNECT initiator. For later streams, the opener is the sender of STREAM_OPEN.

For Fixed streams, Size Class is the exact GDP Size Class. For Variable streams, Size Class is the maximum allowed class and each data packet chooses a class not greater than that maximum.

See [[GTS Stream Profiles]].

## Reliable DATA

Reliable streams are always sequenced. `Sequenced` and `Unchecked Payload` profile bits MUST both be zero for reliable streams.

Reliable Fixed DATA:

```text
Version/Type          1 B
Tunnel ID             4 B
Stream ID             1 B
Sequence              4 B
Data                   N
CRC-32                4 B
```

Reliable Variable DATA adds a 16-bit Valid Length between Sequence and Data.

DATA_END is used only by reliable streams and always includes Valid Length. It consumes a normal sequence number and indicates that no later DATA/DATA_END message will be generated in that sending direction.

Sequence numbers count message packets, not bytes.

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

ACK Base cumulatively acknowledges every reliable message through that sequence. The 32-bit bitmap selectively reports the next 32 sequences. Receive Credit counts additional message packets the receiver guarantees it can accept.

For Reliable Variable streams, one credit guarantees capacity for one packet up to the negotiated maximum Size Class.

Reliable streams use selective retransmission plus an adaptive retransmission timeout. Exact timing constants remain to be frozen.

## Unreliable DATAGRAM

Unreliable streams use DATAGRAM and have no ACK, retransmission, or GTS receive credit.

Unsequenced Fixed:

```text
Version/Type          1 B
Tunnel ID             4 B
Stream ID             1 B
Data                   N
CRC-32                4 B
```

Unsequenced Variable adds Valid Length after Stream ID.

Sequenced Fixed adds a 32-bit Sequence after Stream ID.

Sequenced Variable carries Sequence followed by Valid Length.

Sequenced unreliable streams start Sequence at zero independently in each permitted sending direction and increment by one per DATAGRAM. Missing sequences indicate loss but do not cause retransmission. Duplicate or stale DATAGRAM packets MUST NOT be delivered.

Unsequenced unreliable streams make no ordering, duplicate-suppression, or loss-detection guarantee.

## Payload CRC coverage

Every GTS data/control packet retains a CRC-32 field.

For reliable streams and for unreliable streams with `Unchecked Payload = 0`, CRC-32 covers the canonical GDP pseudo-header plus the complete GTS header, application payload and padding.

For unreliable streams with `Unchecked Payload = 1`, CRC-32 still protects all GTS transport metadata but excludes application payload bytes and padding. Protected metadata includes:

```text
canonical GDP pseudo-header
GTS Version/Type
Tunnel ID
Stream ID
Sequence, when present
Valid Length, when present
```

This mode may deliver damaged application payload but MUST NOT accept corrupted transport metadata as valid. `Unchecked Payload = 1` is invalid on reliable streams.

## CONNECT and Stream 0

CONNECT selects exactly one CSS, creates the tunnel, and creates Stream 0:

```text
Version/Type              1 B
Initiator Receive Tunnel  4 B
Initiator Reset ID        4 B
Stream-0 Profile          2 B
Initial Receive Credit    1 B
CSS Representation/Res.   1 B
CSS Selector              1/4/16 B
Padding                   P
CRC-32                    4 B
```

For an unreliable Stream 0, Initial Receive Credit MUST be zero.

For reliable Stream 0, credit is meaningful only in application-data directions permitted by Direction. A side that cannot receive application data on that stream advertises zero credit.

The CSS representation field is defined by [[Canonical Service Selector]]. Successful CONNECT binds the canonical CSS to the tunnel for its lifetime.

CONNECT_ACK returns responder Tunnel ID, responder Reset ID, status, and initial receive credit. Status `6` means unsupported Stream-0 profile.

## Additional streams

STREAM_OPEN contains:

```text
Version/Type           1 B
Tunnel ID              4 B
Stream ID              1 B
Stream Profile         2 B
Initial Receive Credit 1 B
Reserved               1 B
Padding                P
CRC-32                 4 B
```

STREAM_ACK accepts/rejects the proposed profile and returns initial credit for reliable receive directions. Status `6` means unsupported stream profile.

STREAM_OPEN does not carry CSS and cannot change service identity. A different service requires another CONNECT/tunnel.

## Direction rules

Direction limits only application DATA/DATAGRAM transmission.

Transport control packets such as ACK, STREAM_CLOSE, STREAM_CLOSE_ACK, STREAM_RESET and STREAM_RESET_ACK may be sent in either direction as required by protocol operation.

For reliable unidirectional streams, the endpoint that cannot receive application data MUST advertise zero receive credit.

For all unreliable streams, both sides advertise zero GTS receive credit.

## Graceful stream close

STREAM_CLOSE is directional.

For reliable streams, Final Sequence identifies the final reliable message packet and STREAM_CLOSE_ACK confirms receipt through that sequence.

For unreliable streams, Final Sequence MUST be zero. STREAM_CLOSE/STREAM_CLOSE_ACK only synchronize retirement of stream state and do not imply delivery of prior datagrams.

## STREAM_RESET

STREAM_RESET immediately terminates one stream in both directions while leaving the tunnel and all sibling streams intact.

```text
Version/Type          1 B
Tunnel ID             4 B
Stream ID             1 B
Reason                1 B
Reserved              1 B
CRC-32                4 B
```

Initial reasons:

| Value | Meaning |
|---:|---|
| `0` | unspecified |
| `1` | protocol violation |
| `2` | application abort |
| `3` | resource failure |
| `4` | unsupported/invalid stream state |
| `5` | timeout |
| `6`-`255` | reserved |

The receiver discards outstanding retransmission state, ACK state, receive state and queued undelivered messages for that stream, retires the stream, and replies with STREAM_RESET_ACK carrying Tunnel ID, Stream ID, Reason, Reserved and CRC-32 in the same layout.

STREAM_RESET is retransmitted if its ACK is not received. Repeated STREAM_RESET packets for an already retired stream are idempotent during the stale-state guard interval.

No per-stream Reset ID exists in the baseline.

## Tunnel close and RESET

After all streams are retired, TUNNEL_CLOSE/TUNNEL_CLOSE_ACK gracefully retire the tunnel.

RESET immediately destroys the entire tunnel and all streams. It carries the receiver-local Reset ID established during CONNECT/CONNECT_ACK and is not acknowledged.

Retired Tunnel IDs and Stream IDs are protected by stale-state guard time before reuse.

## End-to-end integrity

CRC-32-GNET parameters remain:

```text
polynomial  0x04C11DB7
init        0xFFFFFFFF
refin       false
refout      false
xorout      0xFFFFFFFF
check       "123456789" -> 0xFC891918
```

The canonical GDP pseudo-header includes GDP Version, GDP Type=`0x2`, GDP Size Class and effective 64-bit Source/Destination identities. Hop Limit and Local/Global representation are excluded.

For full-coverage packets, CRC input continues through the complete GTS header/content/padding. For Unchecked Payload DATAGRAM, payload and padding are omitted from CRC input as defined above.

## Service selection

Service selection is CONNECT-only and uses [[Canonical Service Selector]]. The GDP address identifies the endpoint; CSS identifies the service. Established DATA/DATAGRAM packets carry neither CSS nor TCP-style port numbers.

## Explicitly deferred

- flow-ID/virtual-circuit routing optimization;
- multicast/group transport;
- end-to-end congestion control;
- cryptographic authentication/encryption;
- dynamic routing behavior.

## Remaining work

- exact delayed-ACK/RTO constants for reliable streams;
- exact STREAM_RESET retransmission timing;
- detailed malformed-control error mapping;
- golden packet/CRC/conformance vectors.
