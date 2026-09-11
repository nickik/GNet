---
id: gts-protocol
title: "GTS Protocol"
aliases: ["GTS","GNet Transport and Session Protocol"]
type: protocol
status: frozen
layers: ["L4","L5"]
tags: ["gnet","gnet/protocol","gnet/status/frozen","gnet/layer/l4","gnet/layer/l5"]
parent: "[[Protocols MOC]]"
related: ["[[GTS Transport Packets]]","[[Transport and Flows]]","[[ADR-0005 Tunnels and Streams]]","[[ADR-0018 GDP Header CRC and Local 16-bit Form]]"]
updated: 2026-09-11
---
# GNet Transport and Session Protocol (GTS)

> [!info] Knowledge graph
> **Up:** [[Protocols MOC]] · [[GTS Transport Packets]] · [[Transport and Flows]]

Status: **FROZEN initial traditional packet-routed protocol; timing constants and conformance vectors remain**

GTS is the endpoint transport/session protocol above GDP. The initial profile is deliberately narrow: reliable ordered streams carried in independently routed GDP packets. Flow-ID routing, multicast/group transport, unreliable streams, congestion control, and cryptographic negotiation are deferred.

## Packet types

The GTS first byte contains Version in the high four bits and Type in the low four bits. Initial GTS Version is `0`.

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
| `0xD`-`0xF` | RESERVED |

Unknown/reserved types are discarded.

## Identifiers and sequencing

- Tunnel ID: 32 bits, receiver-local.
- Reset ID: 32 bits, receiver-local capability used only for RESET.
- Stream ID: 8 bits, scoped to one tunnel.
- Packet Sequence: 32 bits, scoped to one stream and counting DATA/DATA_END packets rather than bytes.
- CONNECT initiator owns even Stream IDs; responder owns odd Stream IDs; Stream 0 is created by CONNECT.

A CONNECT exchange establishes one receiver-local Tunnel ID and Reset ID for each endpoint. Ordinary packets carry only the peer-assigned destination Tunnel ID.

## DATA

Ordinary DATA has 14 bytes fixed GTS overhead:

```text
Version/Type          1 B
Tunnel ID             4 B
Stream ID             1 B
Sequence              4 B
Data                   N
CRC-32                4 B
```

DATA never piggybacks ACK state and contains no payload-length field.

## DATA_END

A partial final transport unit uses DATA_END:

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

Valid Length is an unsigned 16-bit number of meaningful application bytes. DATA_END consumes a normal sequence number and is acknowledged exactly like DATA. No later DATA/DATA_END may be generated in that sending direction.

## ACK and reliability

ACK is a separate 20-byte GTS structure:

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

ACK Base cumulatively acknowledges every sequence through that value. Bitmap bit 0 reports ACK Base+1 and bit 31 reports ACK Base+32; `1` means received correctly. Receive Credit is an 8-bit count of additional DATA packets the receiver can accept.

The receiver normally ACKs every two correctly received DATA/DATA_END packets, uses a short delayed-ACK timer when one packet remains pending, and ACKs immediately on a newly observed gap, a gap fill that advances ACK Base, or a credit transition needed to stop/restart sending.

A bitmap-visible hole is retransmitted immediately. A retransmission timeout remains mandatory for losses not exposed by later packets. RTO is adaptive from measured RTT, with one retransmission timer per stream. Exact integer estimator constants and bounds remain to be frozen.

## CONNECT and Stream 0

CONNECT creates the tunnel and Stream 0 in one exchange. It carries:

```text
Version/Type              1 B
Initiator Receive Tunnel  4 B
Initiator Reset ID        4 B
Stream-0 Size Class       1 B
Initial Receive Credit    1 B
Selector Class/Reserved   1 B
Service Selector          variable
Padding                   P
CRC-32                    4 B
```

CONNECT_ACK carries:

```text
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

On accepted CONNECT, the responder returns the Tunnel ID that the initiator subsequently uses when sending to it. On rejection, responder Tunnel ID and Reset ID are zero and no tunnel is established.

Initial CONNECT_ACK statuses are: `0` accepted, `1` service unavailable, `2` resource unavailable, `3` unsupported selector, `4` unsupported Stream-0 Size Class, `5` administratively rejected; all others reserved.

## Additional streams

STREAM_OPEN carries:

```text
Version/Type             1 B
Tunnel ID                4 B
Stream ID                1 B
Data Size Class          1 B
Initial Receive Credit   1 B
Reserved                 1 B
Padding                  P
CRC-32                   4 B
```

STREAM_ACK carries:

```text
Version/Type             1 B
Tunnel ID                4 B
Stream ID                1 B
Status                   1 B
Initial Receive Credit   1 B
Reserved                 1 B
Padding                  P
CRC-32                   4 B
```

Initial STREAM_ACK statuses are: `0` accepted, `1` invalid/wrong-parity Stream ID, `2` Stream ID already in use, `3` resource unavailable, `4` unsupported Data Size Class, `5` administratively rejected; all others reserved.

The initial profile assigns one fixed GDP DATA Size Class to each stream, used in both directions.

## Graceful close

STREAM_CLOSE is directional:

```text
Version/Type          1 B
Tunnel ID             4 B
Stream ID             1 B
Final Sequence        4 B
Padding                P
CRC-32                4 B
```

STREAM_CLOSE_ACK repeats Tunnel ID, Stream ID, and Final Sequence. The close is complete for that direction only after all packets through Final Sequence are received correctly. The opposite direction may remain open.

After all streams are fully retired, the tunnel may close with minimal TUNNEL_CLOSE / TUNNEL_CLOSE_ACK packets containing Version/Type, Tunnel ID, padding, and CRC-32.

## RESET

RESET immediately destroys a tunnel and every stream:

```text
Version/Type          1 B
Tunnel ID             4 B
Reset ID              4 B
Reason                1 B
Padding                P
CRC-32                4 B
```

The Reset ID must match the receiver-local value supplied during CONNECT/CONNECT_ACK. Wrong Reset ID means discard without state change. RESET is not acknowledged and is idempotent during stale-state retention.

Reasons: `0` unspecified fatal failure, `1` protocol/state violation, `2` application/service abort, `3` local resource failure, `4` stale/invalid stream state; others reserved.

## Identifier reuse

Retired Tunnel IDs and Stream IDs are not immediately reused. Implementations retain stale-state rejection information for at least twice the maximum configured retransmission timeout. Packets from a retired incarnation are discarded during that interval.

## End-to-end integrity

GDP protects only its own immutable header information with CRC-8; it does not protect the GDP payload. GTS therefore provides mandatory end-to-end integrity for GTS content. Ordinary GTS uses CRC-32-GNET; a future compact form using the dedicated 0-byte/3-byte GDP classes uses CRC-8-GNET.

CRC-8-GNET: polynomial `0x07`, init `0x00`, no reflection, xorout `0x00`, check `123456789 -> 0xF4`.

CRC-32-GNET: polynomial `0x04C11DB7`, init `0xFFFFFFFF`, no reflection, xorout `0xFFFFFFFF`, transmitted most-significant byte first, check `123456789 -> 0xFC891918`.

CRC input is the canonical GDP pseudo-header followed by all defined GTS content and zero padding. Canonical GDP context includes GDP Version, GDP Type=`0x2`, GDP Size Class, effective 64-bit Source Address, and effective 64-bit Destination Address. Local GDP IDs are expanded to canonical 64-bit endpoint identities first. Hop Limit and local/global representation are excluded.

## Encoding rules

Multi-byte integers are transmitted most-significant byte first. Unused bytes between defined content and CRC are zero and covered by CRC. Reserved fields are transmitted zero and ignored on receipt. Padding is never application data.

## Service selection

Service selection is setup-only. Selector classes remain: `00` = 8-bit registered code; `01` = 32-bit short textual selector; `10` = 128-bit long/private selector; `11` reserved. Exact textual alphabet/packing remains to be frozen separately.

## Explicitly deferred

- flow-ID/virtual-circuit routing optimization;
- multicast/group transport;
- unreliable stream profiles;
- end-to-end congestion control;
- cryptographic authentication/encryption;
- dynamic routing behavior.

## Remaining work

The initial packet/control wire architecture is frozen. Remaining closure work is limited to exact delayed-ACK/RTO numerical constants, textual Service Selector character packing, detailed malformed-control error mappings, and golden packet/CRC/conformance vectors.
