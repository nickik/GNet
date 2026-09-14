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

Status: **FROZEN baseline wire profile; timing constants and golden vectors remain to be added**

GTS uses 32-bit receiver-local Tunnel IDs and 8-bit Stream IDs. A tunnel may contain Reliable Fixed, Reliable Variable, Unreliable Fixed, and Unreliable Variable streams as defined by [[GTS Stream Profiles]]. Reliable streams use packet sequence numbers, selective-repeat ACKs, receive credit and retransmission. Unreliable streams use independent DATAGRAM packets with no GTS ACK or retransmission.

## Packet type registry

The GTS Type is the low four bits of the first GTS byte; the high four bits are GTS Version. Initial GTS Version is `0`.

| Type | Name | Meaning |
|---:|---|---|
| `0x0` | RESERVED | invalid/unassigned |
| `0x1` | CONNECT | select service, create tunnel and Stream 0 |
| `0x2` | CONNECT_ACK | accept/reject CONNECT and return responder state |
| `0x3` | STREAM_OPEN | create an additional stream in the existing service-bound tunnel |
| `0x4` | STREAM_ACK | accept/reject STREAM_OPEN |
| `0x5` | DATA | reliable stream data |
| `0x6` | ACK | reliable-stream cumulative/selective acknowledgement and receive credit |
| `0x7` | DATA_END | final reliable data unit in one sending direction |
| `0x8` | STREAM_CLOSE | graceful directional stream close |
| `0x9` | STREAM_CLOSE_ACK | confirm directional stream close |
| `0xA` | TUNNEL_CLOSE | graceful tunnel close after streams retire |
| `0xB` | TUNNEL_CLOSE_ACK | confirm graceful tunnel close |
| `0xC` | RESET | immediate abnormal tunnel termination |
| `0xD` | DATAGRAM | unreliable stream datagram |
| `0xE`-`0xF` | RESERVED | future use |

Unknown or reserved packet types are discarded.

## Common encoding rules

- multi-byte integers are transmitted most-significant byte first;
- reserved fields are transmitted zero and ignored on receipt unless a future version defines them;
- the enclosing GDP Size Class determines the total GDP payload budget;
- zero padding fills unused bytes before the CRC where a packet uses Valid Length;
- padding is included in the GTS CRC and is never application data;
- baseline GTS packets use CRC-32-GNET.

## Stream Parameters byte

CONNECT and STREAM_OPEN use the same byte:

```text
bit  7      Unreliable
bit  6      Variable
bits 5..4   Reserved = 0
bits 3..0   Size Class
```

Interpretation:

```text
Unreliable=0 Variable=0   Reliable Fixed
Unreliable=0 Variable=1   Reliable Variable
Unreliable=1 Variable=0   Unreliable Fixed
Unreliable=1 Variable=1   Unreliable Variable
```

For Fixed streams, Size Class is the exact GDP Size Class used by every data packet. For Variable streams, Size Class is the maximum class and each data packet may choose any supported class not greater than that maximum.

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

Fixed overhead is 14 bytes. The packet's GDP Size Class is the exact class negotiated for the stream, so no payload-length field is needed. Sequence numbers count DATA/DATA_END packets, not bytes.

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

Fixed overhead is 16 bytes. The sender may choose any GDP Size Class up to the stream maximum. `Valid Length` is the number of meaningful application bytes following the field. Remaining bytes before CRC are zero padding.

Different packet sizes do not change sequence semantics; the sequence still advances by one per DATA/DATA_END packet.

## DATA_END

DATA_END is used only on reliable streams and is sequenced/acknowledged exactly like DATA:

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

For Reliable Fixed, DATA_END provides the partial final unit. For Reliable Variable, it has the same payload layout as variable DATA but additionally states that no later DATA/DATA_END will be generated in that sending direction.

## Unreliable Fixed DATAGRAM

```text
DATAGRAM
--------------------------------
Version/Type          1 B
Tunnel ID             4 B
Stream ID             1 B
Data                   N
CRC-32                4 B
```

Fixed overhead is 10 bytes. Every packet uses the stream's exact GDP Size Class.

DATAGRAM contains no GTS sequence number. GTS performs no acknowledgement, retransmission, reordering, duplicate suppression, or loss detection for unreliable streams.

## Unreliable Variable DATAGRAM

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

Fixed overhead is 12 bytes. The sender chooses any GDP Size Class up to the negotiated stream maximum. `Valid Length` identifies meaningful application bytes and remaining bytes before CRC are zero padding.

Applications that need media sequence numbers, timestamps, epochs or application-specific loss detection place those fields in their own payload.

## ACK packet

ACK exists only for reliable streams:

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

`ACK Base` cumulatively acknowledges every sequence through that value. Bitmap bit 0 represents ACK Base+1 and bit 31 represents ACK Base+32; `1` means the complete packet was received correctly.

`Receive Credit` is a packet count. On a Reliable Fixed stream, one credit reserves one packet of the fixed class. On a Reliable Variable stream, one credit guarantees capacity for one packet up to the negotiated maximum Size Class.

An ACK MUST NOT be generated for an unreliable stream.

## ACK timing and retransmission

Reliable streams retain the baseline selective-repeat behavior: normally ACK every two correctly received DATA/DATA_END packets, use a short delayed ACK when one packet remains pending, immediately report a newly observed gap or a gap fill that advances ACK Base, and retransmit bitmap-visible holes immediately. An adaptive retransmission timeout covers losses not exposed by later packets.

Unreliable streams have no GTS retransmission timer.

## CONNECT

CONNECT selects one CSS, creates the tunnel, and creates Stream 0. Stream 0 may use any of the four baseline stream profiles.

```text
CONNECT
--------------------------------
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

`Stream-0 Parameters` uses the Stream Parameters layout above. For an unreliable Stream 0, `Initial Receive Credit` MUST be zero.

The CSS representation field uses the high two bits:

| Bits | Name | Selector bytes |
|---:|---|---:|
| `00` | Registered-8 | 1 |
| `01` | Short-32 | 4 |
| `10` | Full-128 | 16 |
| `11` | Reserved | — |

The complete namespace and canonicalization rules are defined by [[Canonical Service Selector]]. Successful CONNECT binds the canonical CSS to the tunnel for its lifetime.

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

On success, `Responder Receive Tunnel` is the receiver-local Tunnel ID the initiator uses for packets sent toward the responder. For an unreliable Stream 0, `Initial Receive Credit` MUST be zero.

Status values:

| Status | Meaning |
|---:|---|
| `0` | accepted |
| `1` | service unavailable |
| `2` | resource unavailable |
| `3` | unsupported, malformed, or non-canonical CSS |
| `4` | unsupported Stream-0 Size Class |
| `5` | administratively rejected |
| `6` | unsupported Stream-0 profile |
| `7`-`255` | reserved |

On rejection, responder Tunnel ID and Reset ID are zero and no tunnel state is established.

## Additional streams

Stream IDs are 8 bits with parity ownership:

- CONNECT initiator allocates even Stream IDs;
- CONNECT responder allocates odd Stream IDs;
- Stream 0 is created by CONNECT.

Additional streams belong to the CSS selected by CONNECT.

### STREAM_OPEN

```text
STREAM_OPEN
--------------------------------
Version/Type           1 B
Tunnel ID              4 B
Stream ID              1 B
Stream Parameters      1 B
Initial Receive Credit 1 B
Reserved               1 B
Padding                P
CRC-32                 4 B
```

For an unreliable stream, `Initial Receive Credit` MUST be zero.

### STREAM_ACK

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

Status values:

| Status | Meaning |
|---:|---|
| `0` | accepted |
| `1` | Stream ID invalid or wrong parity |
| `2` | Stream ID already in use |
| `3` | resource unavailable |
| `4` | unsupported Size Class |
| `5` | administratively rejected |
| `6` | unsupported stream profile |
| `7`-`255` | reserved |

For an accepted unreliable stream, `Initial Receive Credit` MUST be zero.

## Graceful stream close

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

```text
STREAM_CLOSE_ACK
--------------------------------
Version/Type          1 B
Tunnel ID             4 B
Stream ID             1 B
Final Sequence        4 B
Padding                P
CRC-32                4 B
```

For a reliable stream, `Final Sequence` identifies the final DATA/DATA_END packet and ACK confirms receipt through it. For an unreliable stream, `Final Sequence` MUST be zero; STREAM_CLOSE/STREAM_CLOSE_ACK only synchronize stream-state retirement and do not imply delivery of previous DATAGRAM packets.

## Graceful tunnel close

A tunnel may be gracefully closed after all streams are retired:

```text
TUNNEL_CLOSE
--------------------------------
Version/Type          1 B
Tunnel ID             4 B
Padding                P
CRC-32                4 B
```

TUNNEL_CLOSE_ACK has the same fields. The receiver retires the tunnel while retaining stale-state protection as required by the protocol.

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

RESET immediately destroys the tunnel and all streams. The Reset ID must match the receiver-local capability established by CONNECT/CONNECT_ACK. RESET is not acknowledged.

## Identifier reuse

Retired Tunnel IDs and Stream IDs are not immediately reused. Implementations retain stale-state rejection information for at least twice the maximum configured retransmission timeout after graceful retirement or RESET.

## Mandatory integrity trailer

The baseline currently retains CRC-32-GNET for reliable DATA, unreliable DATAGRAM, and all ordinary GTS control packets:

```text
CRC-32-GNET
  polynomial  0x04C11DB7
  init        0xFFFFFFFF
  refin       false
  refout      false
  xorout      0xFFFFFFFF
  byte order  most-significant byte first
  check       "123456789" -> 0xFC891918
```

CRC input is the canonical GDP pseudo-header followed by the complete GTS header, defined content, and zero padding. The CRC trailer itself is excluded.

```text
GDP Version
GDP Type = 0x2 (GTS)
GDP Size Class
effective 64-bit Source Address
effective 64-bit Destination Address
complete GTS header/content/padding
```

For Local GDP, local IDs are expanded to their canonical 64-bit endpoint identities before CRC calculation. Hop Limit, Local/Global representation, reserved GDP bits, and mutable forwarding state are excluded.

A failed CRC causes the GTS packet to be discarded. For reliable streams the normal retransmission machinery can recover it; for unreliable streams it is simply lost.

Alternative header-only/no-payload integrity coverage for unreliable media remains a separate design decision and is not frozen here.

## Canonical Service Selector summary

CONNECT follows [[Canonical Service Selector]]. Examples:

```text
<GDP-address>:-FILE
<GDP-address>:-GRPC
<GDP-address>:#CAPI
<GDP-address>:0123456789ABCDEF0123456789ABCDEF
```

CSS is not repeated in DATA, DATAGRAM, ACK, STREAM_OPEN, or other established-tunnel packets.

## Remaining initial-profile work

- exact delayed-ACK timer and adaptive-RTO constants/bounds for reliable streams;
- exact stale-state timing profiles beyond the minimum rule;
- malformed-control-packet reason mapping where not already covered by GCTL;
- golden packet/CRC/conformance vectors;
- decide whether unreliable streams need an optional header-only integrity profile.
