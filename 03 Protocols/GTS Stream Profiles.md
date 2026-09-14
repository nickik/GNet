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

Status: **FROZEN reliability and packet-size profile model**

Every GTS stream has two independent properties:

1. delivery mode: **reliable** or **unreliable**;
2. packet-size mode: **fixed** or **variable**.

This creates exactly four baseline stream profiles:

| Unreliable | Variable | Profile | Data packet | GTS ACK/retransmit |
|---:|---:|---|---|---|
| `0` | `0` | Reliable Fixed | `DATA` | yes |
| `0` | `1` | Reliable Variable | `DATA` + Valid Length | yes |
| `1` | `0` | Unreliable Fixed | `DATAGRAM` | no |
| `1` | `1` | Unreliable Variable | `DATAGRAM` + Valid Length | no |

A tunnel may contain streams using different profiles. Stream 0, created by CONNECT, may use any baseline profile.

## Stream Parameters byte

CONNECT and STREAM_OPEN carry one `Stream Parameters` byte:

```text
bit  7      Unreliable
bit  6      Variable
bits 5..4   Reserved = 0
bits 3..0   Size Class
```

Semantics:

```text
Unreliable = 0   reliable ordered stream
Unreliable = 1   unreliable datagram stream

Variable   = 0   Size Class is exact for every data packet
Variable   = 1   Size Class is the maximum allowed class; each packet chooses its class
```

Keeping both high bits zero therefore preserves the original reliable-fixed interpretation.

## Reliable Fixed

Reliable Fixed is the original GTS data profile.

Every DATA/DATA_END packet uses the exact GDP Size Class negotiated for the stream. Ordinary DATA carries no Valid Length because the stream profile plus GDP Size Class determines the application-data capacity.

Reliable Fixed uses:

- 32-bit packet sequence numbers;
- cumulative/selective ACK;
- receive credit;
- retransmission;
- adaptive retransmission timeout;
- mandatory end-to-end integrity.

## Reliable Variable

Reliable Variable retains the same packet-oriented sequencing and ACK model but permits each DATA/DATA_END packet to choose any GDP Size Class not larger than the stream's negotiated maximum.

Ordinary DATA therefore includes a 16-bit `Valid Length`:

```text
Version/Type          1 B
Tunnel ID             4 B
Stream ID             1 B
Sequence              4 B
Valid Length          2 B
Data                   N
Padding                P
CRC-32                4 B
```

`Valid Length` gives the number of meaningful application bytes after the field. Remaining bytes before CRC are zero padding.

Packet sequence numbers still count packets, not bytes. Different packet sizes therefore do not change ACK numbering.

Receive Credit remains packet-count based. One advertised credit on a Reliable Variable stream guarantees capacity for one packet up to the negotiated maximum Size Class. This deliberately favors simple bounded accounting over byte-granular receive windows.

## Unreliable Fixed

Unreliable Fixed carries independent application datagrams using one exact GDP Size Class.

It uses GTS packet type `DATAGRAM` and has no sequence field:

```text
Version/Type          1 B
Tunnel ID             4 B
Stream ID             1 B
Data                   N
CRC-32                4 B
```

Fixed GTS overhead is 10 bytes.

GTS provides no acknowledgement, retransmission, duplicate suppression, ordering, or loss detection for these packets.

## Unreliable Variable

Unreliable Variable permits each DATAGRAM packet to choose any GDP Size Class up to the stream's negotiated maximum.

Because the selected GDP class may exceed the amount of application data, the packet carries a 16-bit Valid Length:

```text
Version/Type          1 B
Tunnel ID             4 B
Stream ID             1 B
Valid Length          2 B
Data                   N
Padding                P
CRC-32                4 B
```

Fixed GTS overhead is 12 bytes.

There is no GTS sequence number, ACK, receive credit, retransmission, duplicate suppression, or reordering. Applications that need timestamps, media sequence numbers, epochs, or application-specific loss detection carry those fields in their own payload.

## GTS receive credit

GTS Receive Credit applies only to reliable streams.

For unreliable streams:

- Initial Receive Credit in CONNECT/CONNECT_ACK or STREAM_OPEN/STREAM_ACK MUST be zero;
- ACK packets MUST NOT be generated for the stream;
- a receiver that cannot accept a valid DATAGRAM may discard it.

This does not remove lower-layer GNet hop-local credit/backpressure. GTS unreliable delivery is unreliable at the end-to-end transport layer; adjacent GNet forwarding endpoints still obey the normal link/data-path capacity rules.

## ACK behavior

The existing ACK format is unchanged and valid only for reliable streams:

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

Reliable Fixed and Reliable Variable both use the same packet-oriented sequence and selective-repeat ACK model.

Unreliable Fixed and Unreliable Variable never generate GTS ACKs for DATAGRAM delivery.

## DATA_END and stream close

Reliable streams use DATA_END as the sequenced final partial data packet in one sending direction.

Unreliable streams do not use DATA_END. Every DATAGRAM is independently bounded by GDP Size Class and, for Variable mode, Valid Length.

STREAM_CLOSE remains the directional state-retirement operation for all stream profiles:

- reliable stream: `Final Sequence` identifies the final reliable packet and STREAM_CLOSE_ACK confirms receipt through that sequence;
- unreliable stream: `Final Sequence` MUST be zero and has no delivery meaning; STREAM_CLOSE/STREAM_CLOSE_ACK only synchronizes retirement of stream state and does not imply delivery of prior datagrams.

## Stream establishment

CONNECT creates Stream 0 using the supplied Stream Parameters byte.

STREAM_OPEN creates an additional stream using the same Stream Parameters format.

The receiver may reject an unsupported profile. CONNECT_ACK and STREAM_ACK status `6` means **unsupported stream profile**.

The accepted profile is symmetric for the stream in the baseline: both directions use the same reliability mode, fixed/variable rule, and Size Class or maximum Size Class.

## Integrity

The baseline defined here retains the existing GTS CRC-32-GNET integrity trailer for reliable and unreliable data packets. Unreliable means no delivery/retransmission guarantee; it does not mean unchecked corruption.

Alternative integrity coverage for unreliable media is a separate design decision and is not defined by this profile document.
