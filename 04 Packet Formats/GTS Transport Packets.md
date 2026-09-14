---
id: gts-transport-packets
title: "GTS Transport Packets"
aliases: ["GTS packets"]
type: packet
status: frozen
layers: ["L4","L5"]
tags: ["gnet","gnet/packet","gnet/status/frozen","gnet/layer/l4","gnet/layer/l5"]
parent: "[[Packet Formats MOC]]"
related: ["[[GTS Protocol]]","[[GTS Stream Profiles]]","[[Canonical Service Selector]]","[[ADR-0005 Tunnels and Streams]]","[[GDP Datagram]]"]
updated: 2026-09-14
---
# GTS transport packets

Status: **FROZEN extended stream wire profile; timing constants and golden vectors remain**

## Packet type registry

| Type | Name | Meaning |
|---:|---|---|
| `0x0` | RESERVED | invalid/unassigned |
| `0x1` | CONNECT | select service, create tunnel and Stream 0 |
| `0x2` | CONNECT_ACK | accept/reject CONNECT |
| `0x3` | STREAM_OPEN | create additional stream |
| `0x4` | STREAM_ACK | accept/reject STREAM_OPEN |
| `0x5` | DATA | reliable message |
| `0x6` | ACK | reliable message acknowledgement/credit |
| `0x7` | DATA_END | final reliable message in one direction |
| `0x8` | STREAM_CLOSE | graceful directional close |
| `0x9` | STREAM_CLOSE_ACK | confirm directional close |
| `0xA` | TUNNEL_CLOSE | graceful tunnel close |
| `0xB` | TUNNEL_CLOSE_ACK | confirm tunnel close |
| `0xC` | RESET | immediate tunnel reset |
| `0xD` | DATAGRAM | unreliable message |
| `0xE` | STREAM_RESET | immediate one-stream reset |
| `0xF` | STREAM_RESET_ACK | confirm stream reset |

The first GTS byte stores Version in the high four bits and Type in the low four bits. Version `0` is the initial version.

## Common rules

- multi-byte integers are transmitted most-significant byte first;
- reserved fields are transmitted zero;
- every GTS packet retains a CRC-32 field;
- one DATA, DATA_END, or DATAGRAM packet is one application message unit;
- zero padding fills unused bytes where Valid Length is present;
- padding is never delivered to the application.

## Stream Profile field

CONNECT and STREAM_OPEN carry a 16-bit Stream Profile:

```text
15      Unreliable
14      Variable
13      Sequenced
12      Unchecked Payload
11..10  Direction
9..4    Reserved = 0
3..0    Size Class
```

Direction values:

```text
00   reserved/invalid
01   opener -> peer
10   peer -> opener
11   bidirectional
```

For Fixed streams, Size Class is exact. For Variable streams, Size Class is the maximum allowed class.

## Reliable Fixed DATA

```text
DATA
--------------------------------
Version/Type          1 B
Tunnel ID             4 B
Stream ID             1 B
Sequence              4 B
Data                   N
CRC-32                4 B
```

Fixed overhead: 14 bytes.

## Reliable Variable DATA

```text
DATA
--------------------------------
Version/Type          1 B
Tunnel ID             4 B
Stream ID             1 B
Sequence              4 B
Valid Length          2 B
Data                   N
Padding                P
CRC-32                4 B
```

Fixed overhead: 16 bytes.

`Valid Length` is the number of meaningful application bytes following the field.

## DATA_END

```text
DATA_END
--------------------------------
Version/Type          1 B
Tunnel ID             4 B
Stream ID             1 B
Sequence              4 B
Valid Length          2 B
Data                   N
Padding                P
CRC-32                4 B
```

DATA_END is valid only on reliable streams. It is sequenced and acknowledged like DATA and marks the final reliable message in that sending direction.

## Unreliable DATAGRAM forms

### Unsequenced Fixed

```text
DATAGRAM
--------------------------------
Version/Type          1 B
Tunnel ID             4 B
Stream ID             1 B
Data                   N
CRC-32                4 B
```

Fixed overhead: 10 bytes.

### Unsequenced Variable

```text
DATAGRAM
--------------------------------
Version/Type          1 B
Tunnel ID             4 B
Stream ID             1 B
Valid Length          2 B
Data                   N
Padding                P
CRC-32                4 B
```

Fixed overhead: 12 bytes.

### Sequenced Fixed

```text
DATAGRAM
--------------------------------
Version/Type          1 B
Tunnel ID             4 B
Stream ID             1 B
Sequence              4 B
Data                   N
CRC-32                4 B
```

Fixed overhead: 14 bytes.

### Sequenced Variable

```text
DATAGRAM
--------------------------------
Version/Type          1 B
Tunnel ID             4 B
Stream ID             1 B
Sequence              4 B
Valid Length          2 B
Data                   N
Padding                P
CRC-32                4 B
```

Fixed overhead: 16 bytes.

Sequenced unreliable streams increment Sequence by one per DATAGRAM independently in each permitted sending direction. No ACK or retransmission follows.

## ACK

ACK is valid only for reliable streams:

```text
ACK
--------------------------------
Version/Type          1 B
Tunnel ID             4 B
Stream ID             1 B
ACK Base              4 B
Receive Bitmap        4 B
Receive Credit        1 B
Reserved              1 B
CRC-32                4 B
--------------------------------
                      20 B
```

`ACK Base` cumulatively acknowledges every reliable message through that sequence. Bitmap bit 0 represents ACK Base+1 and bit 31 represents ACK Base+32.

Receive Credit is a packet/message count. For Reliable Variable, one credit guarantees capacity for one packet up to the negotiated maximum class.

## CONNECT

```text
CONNECT
--------------------------------
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

Stream-0 Profile uses the 16-bit layout above.

Initial Receive Credit MUST be zero for unreliable Stream 0 and for a reliable endpoint that cannot receive application data under the selected Direction.

CSS encoding follows [[Canonical Service Selector]].

## CONNECT_ACK

```text
CONNECT_ACK
--------------------------------
Version/Type              1 B
Initiator Receive Tunnel  4 B
Responder Receive Tunnel  4 B
Responder Reset ID        4 B
Status                    1 B
Initial Receive Credit    1 B
Reserved                  1 B
Padding                   P
CRC-32                    4 B
```

Status values:

| Status | Meaning |
|---:|---|
| `0` | accepted |
| `1` | service unavailable |
| `2` | resource unavailable |
| `3` | unsupported/malformed CSS |
| `4` | unsupported Size Class |
| `5` | administratively rejected |
| `6` | unsupported Stream-0 profile |
| `7`-`255` | reserved |

## STREAM_OPEN

```text
STREAM_OPEN
--------------------------------
Version/Type           1 B
Tunnel ID              4 B
Stream ID              1 B
Stream Profile         2 B
Initial Receive Credit 1 B
Reserved               1 B
Padding                P
CRC-32                 4 B
```

## STREAM_ACK

```text
STREAM_ACK
--------------------------------
Version/Type           1 B
Tunnel ID              4 B
Stream ID              1 B
Status                 1 B
Initial Receive Credit 1 B
Reserved               1 B
Padding                P
CRC-32                 4 B
```

Status `6` means unsupported stream profile.

For unreliable streams and non-receiving reliable directions, Initial Receive Credit is zero.

## STREAM_CLOSE

```text
STREAM_CLOSE
--------------------------------
Version/Type          1 B
Tunnel ID             4 B
Stream ID             1 B
Final Sequence        4 B
Padding                P
CRC-32                4 B
```

STREAM_CLOSE_ACK repeats Tunnel ID, Stream ID and Final Sequence.

For reliable streams, Final Sequence identifies the final reliable message. For unreliable streams Final Sequence MUST be zero and has no delivery meaning.

## STREAM_RESET

```text
STREAM_RESET
--------------------------------
Version/Type          1 B
Tunnel ID             4 B
Stream ID             1 B
Reason                1 B
Reserved              1 B
CRC-32                4 B
--------------------------------
                     12 B
```

Reasons:

| Value | Meaning |
|---:|---|
| `0` | unspecified |
| `1` | protocol violation |
| `2` | application abort |
| `3` | resource failure |
| `4` | unsupported/invalid stream state |
| `5` | timeout |
| `6`-`255` | reserved |

STREAM_RESET immediately destroys only the named stream, in both directions. Its tunnel and sibling streams remain active.

## STREAM_RESET_ACK

```text
STREAM_RESET_ACK
--------------------------------
Version/Type          1 B
Tunnel ID             4 B
Stream ID             1 B
Reason                1 B
Reserved              1 B
CRC-32                4 B
--------------------------------
                     12 B
```

The ACK confirms peer stream-state retirement. Repeated STREAM_RESET packets are idempotent during the stale-state interval.

## TUNNEL_CLOSE / TUNNEL_CLOSE_ACK

```text
Version/Type          1 B
Tunnel ID             4 B
Padding                P
CRC-32                4 B
```

A graceful tunnel close is valid only after streams are retired.

## RESET

```text
RESET
--------------------------------
Version/Type          1 B
Tunnel ID             4 B
Reset ID              4 B
Reason                1 B
Padding                P
CRC-32                4 B
```

RESET immediately destroys the entire tunnel and all streams.

## CRC-32 coverage

```text
CRC-32-GNET
polynomial  0x04C11DB7
init        0xFFFFFFFF
refin       false
refout      false
xorout      0xFFFFFFFF
check       "123456789" -> 0xFC891918
```

The canonical GDP pseudo-header is always included:

```text
GDP Version
GDP Type = 0x2
GDP Size Class
effective 64-bit Source Address
effective 64-bit Destination Address
```

### Full coverage

Normal packets cover:

```text
canonical GDP pseudo-header
complete GTS header
application payload
zero padding
```

### Unchecked Payload coverage

For an unreliable stream with `Unchecked Payload=1`, CRC input is:

```text
canonical GDP pseudo-header
Version/Type
Tunnel ID
Stream ID
Sequence, if present
Valid Length, if present
```

Application payload and zero padding are excluded. The CRC field remains present and unchanged in size.

This mode is invalid on reliable streams.

## Identifier reuse

Retired Tunnel IDs and Stream IDs are not immediately reused. Implementations retain stale-state rejection information for at least twice the maximum configured retransmission timeout after graceful close or reset.
