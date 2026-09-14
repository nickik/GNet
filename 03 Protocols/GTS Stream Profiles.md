---
id: gts-stream-profiles
title: "GTS Stream Profiles"
aliases: ["GTS stream modes","GTS reliable and unreliable streams"]
type: protocol
status: frozen
layers: ["L4","L5"]
tags: ["gnet","gnet/protocol","gnet/status/frozen","gnet/layer/l4","gnet/layer/l5"]
parent: "[[Protocols MOC]]"
related: ["[[GTS Protocol]]","[[GTS Transport Packets]]","[[Canonical Service Selector]]"]
updated: 2026-09-14
---
# GTS stream profiles

Status: **FROZEN extended stream-profile model**

Every GTS stream has a 16-bit Stream Profile. The profile independently selects reliability, packet sizing, optional sequencing for unreliable traffic, payload CRC coverage, direction, and GDP Size Class.

## Stream Profile

```text
bit 15      Unreliable
bit 14      Variable
bit 13      Sequenced
bit 12      Unchecked Payload
bits 11..10 Direction
bits 9..4   Reserved = 0
bits 3..0   Size Class
```

### Reliability and sizing

```text
Unreliable = 0   reliable ordered message delivery
Unreliable = 1   unreliable datagram delivery

Variable   = 0   Size Class is exact for every data packet
Variable   = 1   Size Class is the maximum allowed class; each packet chooses its class
```

This retains the four baseline combinations:

| Unreliable | Variable | Profile |
|---:|---:|---|
| `0` | `0` | Reliable Fixed |
| `0` | `1` | Reliable Variable |
| `1` | `0` | Unreliable Fixed |
| `1` | `1` | Unreliable Variable |

Stream 0 may use any valid profile. One tunnel may mix different profiles across streams.

## Message preservation

GTS is message-preserving.

One DATA, DATA_END, or DATAGRAM packet carries one application message unit. Reliable delivery preserves both message order and message boundaries. GTS does not merge adjacent application messages into a byte stream and does not split one GTS message across multiple GTS data packets in the baseline profile.

Operating systems and libraries MAY expose a conventional byte-stream API by concatenating delivered reliable messages. That API adaptation does not change the GTS wire semantics.

Applications that need explicit record/message boundaries can therefore use GTS directly without reconstructing boundaries above transport.

## Direction

Direction is relative to the endpoint that opens the stream:

```text
00   reserved / invalid
01   opener -> peer only
10   peer -> opener only
11   bidirectional
```

For Stream 0, the opener is the CONNECT initiator. For later streams, the opener is the sender of STREAM_OPEN.

Direction restricts application DATA/DATAGRAM transmission only. ACK, STREAM_CLOSE, STREAM_CLOSE_ACK, STREAM_RESET, STREAM_RESET_ACK and other transport control packets may travel as required regardless of the application-data direction.

## Reliable streams

Reliable streams use:

- 32-bit packet sequence numbers;
- reliable ordered message delivery;
- cumulative/selective ACK;
- receive credit;
- selective retransmission;
- adaptive retransmission timeout;
- full end-to-end CRC-32 coverage.

`Sequenced` MUST be zero on reliable streams because reliable streams are inherently sequenced.

`Unchecked Payload` MUST be zero on reliable streams.

### Reliable Fixed

Every DATA packet uses the exact negotiated GDP Size Class. Ordinary DATA carries no Valid Length; the packet capacity is implied by the stream profile and GDP Size Class.

DATA_END may carry a shorter final message using Valid Length.

### Reliable Variable

Each DATA packet may use any GDP Size Class not greater than the negotiated maximum. DATA carries a 16-bit Valid Length. Remaining bytes before CRC are zero padding.

Receive Credit remains packet-count based. One credit guarantees capacity for one message packet up to the negotiated maximum Size Class.

## Unreliable streams

Unreliable streams use DATAGRAM and have:

- no GTS ACK;
- no retransmission;
- no GTS receive credit;
- no guaranteed delivery;
- no guaranteed ordering unless the Sequenced option is enabled.

A receiver unable to accept a valid DATAGRAM may discard it.

### Unsequenced unreliable

With `Sequenced = 0`, DATAGRAM carries no Sequence field. GTS does not detect loss, reordering, or duplication.

### Sequenced unreliable

With `Sequenced = 1`, every DATAGRAM carries a 32-bit Sequence field.

Sequence starts at zero independently in each permitted sending direction and increments by one per DATAGRAM. There are still no ACKs or retransmissions.

The receiver uses the sequence to identify newer versus duplicate/stale datagrams. Duplicate or stale datagrams MUST NOT be delivered. Missing sequence numbers indicate loss but do not trigger GTS recovery.

This profile is intended for media, telemetry, state updates, and similar traffic where freshness matters more than recovery.

## Fixed and Variable unreliable

Unreliable Fixed uses one exact GDP Size Class and carries no Valid Length.

Unreliable Variable may choose any GDP Size Class up to the negotiated maximum and carries a 16-bit Valid Length. Remaining bytes before CRC are zero padding.

Sequenced and Variable are independent; all four unreliable combinations are valid.

## Unchecked Payload

`Unchecked Payload = 1` is valid only when `Unreliable = 1`.

The CRC-32 field remains present and MUST still validate all transport metadata. The application payload bytes and zero padding are excluded from CRC calculation.

The protected CRC input still includes:

```text
canonical GDP pseudo-header
GTS Version/Type
Tunnel ID
Stream ID
Sequence, when present
Valid Length, when present
```

The unchecked region is:

```text
application payload
zero padding
```

This allows damaged media payload to be delivered while preventing corruption of tunnel selection, stream selection, packet type, sequence, or payload length from being accepted as valid transport metadata.

`Unchecked Payload = 0` means CRC-32 covers the complete GTS header, application payload, and padding as normal.

Reliable streams MUST use full payload coverage.

## Receive credit and direction

GTS Receive Credit applies only to reliable application-data directions.

For unreliable streams, all Initial Receive Credit fields MUST be zero.

For reliable unidirectional streams:

```text
Direction 01   opener -> peer
               opener receive credit = 0
               peer receive credit may be non-zero

Direction 10   peer -> opener
               opener receive credit may be non-zero
               peer receive credit = 0

Direction 11   both directions
               both sides may advertise receive credit
```

Lower-layer GNet hop-local credit/backpressure remains independent and mandatory.

## Stream establishment

CONNECT creates Stream 0 and carries the Stream Profile. STREAM_OPEN creates an additional stream and carries the same 16-bit Stream Profile format.

The receiver either accepts the proposed profile or rejects it. Baseline GTS does not counter-negotiate individual profile bits.

CONNECT_ACK / STREAM_ACK status `6` means unsupported stream profile.

## Graceful close

STREAM_CLOSE is directional.

For reliable streams, Final Sequence identifies the final reliable message packet in that sending direction. STREAM_CLOSE_ACK confirms receipt through that sequence.

For unreliable streams, Final Sequence MUST be zero and has no delivery meaning. STREAM_CLOSE/STREAM_CLOSE_ACK only synchronize retirement of that direction's state.

## Immediate stream reset

STREAM_RESET immediately terminates one stream in both directions without destroying its tunnel or sibling streams. Outstanding reliable packets, ACK state, receive state, and queued undelivered messages for that stream are discarded.

STREAM_RESET_ACK confirms that the peer has retired the stream. Repeated STREAM_RESET packets for the same retired stream are idempotent during the stale-state guard interval.

No per-stream Reset ID is required; tunnel participation, receiver-local Tunnel ID, Stream ID, GTS CRC and stale-state protection are sufficient for the baseline.
