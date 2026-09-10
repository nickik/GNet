---
id: gts-transport-packets
title: "GTS Transport Packets"
aliases: ["GTS packets"]
type: packet
status: frozen
layers: ["L4","L5"]
tags: ["gnet","gnet/packet","gnet/status/frozen","gnet/layer/l4","gnet/layer/l5"]
parent: "[[Packet Formats MOC]]"
related: ["[[GTS Protocol]]","[[ADR-0005 Tunnels and Streams]]","[[GDP Datagram]]"]
updated: 2026-09-10
---
# GTS transport packets

Status: **FROZEN initial traditional packet-routed wire profile; timing constants and golden vectors remain to be added**

The initial traditional packet-routed GTS profile uses 32-bit receiver-local Tunnel IDs, 8-bit Stream IDs, 32-bit packet sequence numbers, reliable ordered packet delivery, selective-repeat acknowledgement, separate DATA and ACK packets, packet-by-packet GDP routing, and mandatory end-to-end CRC.

Flow-ID routing, multicast/group transport, unreliable streams, transport congestion control, generic options, and cryptographic negotiation are outside this initial profile.

## Packet type registry

The GTS Type is the low four bits of the first GTS byte; the high four bits are GTS Version.

| Type | Name | Meaning |
|---:|---|---|
| `0x0` | RESERVED | invalid/unassigned |
| `0x1` | CONNECT | create tunnel and Stream 0 |
| `0x2` | CONNECT_ACK | accept/reject CONNECT and return responder state |
| `0x3` | STREAM_OPEN | create an additional stream |
| `0x4` | STREAM_ACK | accept/reject STREAM_OPEN |
| `0x5` | DATA | ordinary full DATA unit |
| `0x6` | ACK | cumulative + selective acknowledgement and receive credit |
| `0x7` | DATA_END | final partial DATA unit in one sending direction |
| `0x8` | STREAM_CLOSE | graceful directional stream close |
| `0x9` | STREAM_CLOSE_ACK | confirm directional stream close |
| `0xA` | TUNNEL_CLOSE | graceful tunnel close after streams retire |
| `0xB` | TUNNEL_CLOSE_ACK | confirm graceful tunnel close |
| `0xC` | RESET | immediate abnormal tunnel termination |
| `0xD`-`0xF` | RESERVED | future use |

Unknown or reserved packet types are discarded. An implementation MUST NOT reinterpret an unknown type as DATA or any other known type.

## Common encoding rules

- multi-byte integers are transmitted most-significant byte first;
- reserved fields are transmitted as zero and ignored on receipt unless a future version defines them;
- ordinary GTS packets use CRC-32-GNET;
- the enclosing GDP Size Class determines the total fixed GDP payload budget;
- unused bytes after the defined GTS content and before the CRC are zero padding unless otherwise stated;
- padding is included in the GTS CRC but is never delivered to the application;
- GTS Version `0` is the initial version.

## Ordinary DATA packet

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

Fixed overhead is 14 bytes. Sequence numbers identify DATA packets, not byte offsets, and are interpreted modulo `2^32` independently for each stream.

DATA does not carry acknowledgement state, receive credit, service selector, Reset ID, source Tunnel ID, QoS, flags, or payload length.

## DATA_END packet

`DATA_END` is sequenced and acknowledged exactly like DATA but carries a 16-bit Valid Length before its application bytes:

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

`Valid Length` is the number of application-data bytes following the field that are meaningful. It MUST NOT exceed the remaining data capacity of the enclosing GDP Size Class. Bytes after Valid Length and before CRC are zero padding and are not delivered.

`DATA_END` means that no further DATA/DATA_END sequence numbers will be generated in this sending direction unless a later stream-incarnation mechanism is explicitly defined. The opposite direction remains independent.

## ACK packet

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

`ACK Base` is the highest DATA/DATA_END sequence number such that that packet and every preceding packet in the current sequence space have been received correctly. It cumulatively acknowledges every packet through that sequence.

The 32-bit Receive Bitmap reports the next 32 sequence numbers after ACK Base. Bit 0 represents ACK Base + 1 and bit 31 represents ACK Base + 32. A `1` means the complete packet was received correctly; `0` means it remains missing.

`Receive Credit` is an unsigned 8-bit count of additional DATA/DATA_END packets the receiver is prepared to accept. The unit is packets, not bytes. `0` forbids introduction of new sequence numbers; retransmission of already-outstanding packets does not consume new sequence-space credit. `255` means at least 255 additional packets may be accepted.

## ACK timing and retransmission

Under normal in-order traffic the receiver sends an ACK after every two correctly received DATA/DATA_END packets. If only one packet is pending acknowledgement, it sends an ACK when the short delayed-ACK timer expires.

An ACK is sent immediately when a gap is first observed, when a previously reported gap is filled and ACK Base advances, or when Receive Credit reaches zero or changes in a way required to restart a stalled sender.

When an ACK bitmap positively acknowledges a later packet while an earlier outstanding packet in the represented range remains zero, the sender treats the earlier packet as an explicit hole and retransmits it immediately. GTS does not use duplicate-ACK counting for this purpose.

A retransmission timeout remains mandatory. The initial profile uses an adaptive timeout from measured RTT. One retransmission timer per stream is sufficient. On timeout, the oldest outstanding unacknowledged packet is retransmitted and the timer is restarted conservatively. Exact estimator constants and timer bounds remain to be frozen separately.

## CONNECT

CONNECT creates a tunnel and simultaneously creates Stream 0. The initiator chooses an even Stream ID namespace and Stream 0 belongs to the initiator.

CONNECT layout:

```text
CONNECT
--------------------------------
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

`Initiator Receive Tunnel` is the 32-bit receiver-local Tunnel ID the responder MUST use for packets sent toward the initiator after successful establishment.

`Initiator Reset ID` is a 32-bit receiver-local reset capability allocated by the initiator. A RESET directed at the initiator must present this value.

`Stream-0 Size Class` uses the low four bits to identify the fixed GDP Size Class used for ordinary DATA on Stream 0. The high four bits are reserved and transmitted zero.

`Initial Receive Credit` advertises how many Stream-0 DATA packets the initiator can initially accept from the responder.

`Selector Class/Reserved` uses the top two bits for the Service Selector class and the low six bits as zero/reserved. The selector field immediately follows and has the length implied by the selector class.

The CONNECT packet itself has no destination Tunnel ID because no responder-local tunnel exists yet. GDP source/destination addressing identifies the two endpoints.

## CONNECT_ACK

CONNECT_ACK returns the responder-local state and either accepts or rejects the request.

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

The `Initiator Receive Tunnel` echoes the value supplied in CONNECT and correlates the reply without requiring a separate transaction number.

On success, `Responder Receive Tunnel` is the receiver-local Tunnel ID the initiator MUST use for subsequent packets sent to the responder. `Responder Reset ID` is the reset capability required for a RESET directed at the responder.

`Initial Receive Credit` is the responder's initial receive credit for Stream 0.

Initial Status values:

| Status | Meaning |
|---:|---|
| `0` | accepted |
| `1` | service unavailable |
| `2` | resource unavailable |
| `3` | unsupported selector |
| `4` | unsupported Stream-0 Size Class |
| `5` | administratively rejected |
| `6`-`255` | reserved |

On rejection, the responder receive Tunnel ID and Reset ID MUST be zero and no tunnel state is established.

## Additional streams

Stream IDs are 8 bits with parity ownership:

- CONNECT initiator allocates even Stream IDs;
- CONNECT responder allocates odd Stream IDs;
- Stream 0 is created by CONNECT.

An endpoint MUST NOT allocate a Stream ID owned by the peer's parity.

### STREAM_OPEN

```text
STREAM_OPEN
--------------------------------
Version/Type          1 B
Tunnel ID             4 B
Stream ID             1 B
Data Size Class       1 B
Initial Receive Credit 1 B
Reserved              1 B
Padding               P
CRC-32                4 B
```

`Tunnel ID` is the peer's receiver-local Tunnel ID. `Data Size Class` uses the low four bits; the high four bits are zero/reserved. The stream uses that fixed GDP Size Class for ordinary DATA/DATA_END in both directions in the initial profile.

The opening endpoint's `Initial Receive Credit` states how many packets it can initially receive on the new stream.

### STREAM_ACK

```text
STREAM_ACK
--------------------------------
Version/Type          1 B
Tunnel ID             4 B
Stream ID             1 B
Status                1 B
Initial Receive Credit 1 B
Reserved              1 B
Padding               P
CRC-32                4 B
```

Initial Status values:

| Status | Meaning |
|---:|---|
| `0` | accepted |
| `1` | Stream ID invalid or wrong parity |
| `2` | Stream ID already in use |
| `3` | resource unavailable |
| `4` | unsupported Data Size Class |
| `5` | administratively rejected |
| `6`-`255` | reserved |

On acceptance, `Initial Receive Credit` is the responder's initial receive credit for the stream. On rejection it is zero.

## Graceful stream close

`STREAM_CLOSE` is directional and states that the sender will transmit no more DATA/DATA_END in that direction.

```text
STREAM_CLOSE
--------------------------------
Version/Type          1 B
Tunnel ID             4 B
Stream ID             1 B
Final Sequence        4 B
Padding               P
CRC-32                4 B
```

`Final Sequence` is the sequence number of the final DATA/DATA_END packet in this sending direction. The receiver MUST NOT consider the sending direction gracefully closed until every packet through Final Sequence has been received correctly.

`STREAM_CLOSE_ACK` confirms that condition:

```text
STREAM_CLOSE_ACK
--------------------------------
Version/Type          1 B
Tunnel ID             4 B
Stream ID             1 B
Final Sequence        4 B
Padding               P
CRC-32                4 B
```

The opposite stream direction may remain open. A stream is fully retired only after both directions have completed graceful close.

## Graceful tunnel close

A tunnel may be gracefully closed only after all streams are fully retired.

```text
TUNNEL_CLOSE
--------------------------------
Version/Type          1 B
Tunnel ID             4 B
Padding               P
CRC-32                4 B
```

```text
TUNNEL_CLOSE_ACK
--------------------------------
Version/Type          1 B
Tunnel ID             4 B
Padding               P
CRC-32                4 B
```

The peer receiving TUNNEL_CLOSE replies with TUNNEL_CLOSE_ACK and retires the tunnel after retaining sufficient stale-packet guard state. The initiator retires its side after receiving TUNNEL_CLOSE_ACK.

## RESET

RESET is immediate abnormal termination of the entire tunnel and all of its streams. It is not a graceful close and does not wait for outstanding DATA.

```text
RESET
--------------------------------
Version/Type          1 B
Tunnel ID             4 B
Reset ID              4 B
Reason                1 B
Padding               P
CRC-32                4 B
```

`Tunnel ID` identifies the receiver-local tunnel to destroy. `Reset ID` MUST equal the 32-bit reset capability the receiver supplied during CONNECT/CONNECT_ACK. A RESET with an incorrect Reset ID is discarded without changing tunnel state.

Initial Reason values:

| Reason | Meaning |
|---:|---|
| `0` | unspecified fatal failure |
| `1` | protocol/state violation |
| `2` | application/service abort |
| `3` | local resource failure |
| `4` | stale/invalid stream state |
| `5`-`255` | reserved |

RESET is not acknowledged. Repeated valid RESET packets are idempotent during the stale-packet guard interval.

## Stale packet and identifier reuse baseline

A receiver MUST NOT immediately reuse a recently retired Tunnel ID or Stream ID while delayed packets from the previous incarnation could still be accepted as current traffic.

For the initial profile, implementations maintain a finite stale-state guard interval of at least **twice the maximum configured retransmission timeout** after graceful retirement or RESET. During this interval, packets for the retired incarnation are discarded. Exact larger guard requirements may be defined later for specific long-delay network profiles.

## Mandatory integrity trailer

Every ordinary GTS packet uses CRC-32-GNET except future compact GTS forms using the dedicated 0-byte/3-byte tiny classes, which use CRC-8-GNET.

```text
CRC-8-GNET
  polynomial  0x07
  init        0x00
  refin       false
  refout      false
  xorout      0x00
  check       "123456789" -> 0xF4

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

For Global GDP, effective addresses are the transmitted 64-bit addresses. For Local GDP, the local IDs are expanded to their canonical 64-bit endpoint identities before CRC calculation. Hop Limit, Local/Global representation, reserved GDP bits, and mutable forwarding state are excluded.

A failed CRC causes the packet to be discarded and treated as not received.

## Service Selector

Service selection is setup-only. The initial selector classes remain:

| Class | Selector field | Meaning |
|---:|---:|---|
| `00` | 8 bits | numeric registered service code |
| `01` | 32 bits | short textual selector |
| `10` | 128 bits | long/private textual selector |
| `11` | reserved | future expansion |

The exact textual alphabet/packing remains a separate registry-format decision; it does not change the control packet envelope above.

## Remaining initial-profile work

The core wire layouts and state transitions are now frozen. Remaining work is limited to:

- exact delayed-ACK timer and adaptive-RTO integer constants/bounds;
- exact stale-state timing profiles beyond the minimum rule;
- textual Service Selector alphabet/packing;
- malformed-control-packet reason mapping where not already covered by GCTL;
- golden packet/CRC vectors and conformance tests.
