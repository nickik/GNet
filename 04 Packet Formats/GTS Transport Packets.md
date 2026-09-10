---
id: gts-transport-packets
title: "GTS Transport Packets"
aliases: ["GTS packets"]
type: packet
status: open
layers: ["L4","L5"]
tags: ["gnet","gnet/packet","gnet/status/open","gnet/layer/l4","gnet/layer/l5"]
parent: "[[Packet Formats MOC]]"
related: ["[[GTS Protocol]]","[[ADR-0005 Tunnels and Streams]]","[[GDP Datagram]]"]
updated: 2026-09-10
---
# GTS transport packets

Status: **FROZEN ordinary DATA/ACK reliability and lifecycle semantics; OPEN exact connection/control packing**

The initial traditional packet-routed GTS profile uses:

- 32-bit receiver-local Tunnel IDs;
- 8-bit Stream IDs scoped to one tunnel;
- 32-bit packet sequence numbers scoped to one stream;
- reliable ordered packet delivery within each stream;
- selective-repeat acknowledgement;
- separate DATA and ACK packet types;
- packet-by-packet GDP routing;
- mandatory end-to-end CRC.

Flow-ID routing, multicast/group transport, unreliable streams, and transport congestion control are outside this initial profile.

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

A partial final DATA unit uses the distinct `DATA_END` packet type rather than changing ordinary DATA.

`DATA_END` carries the same Tunnel ID, Stream ID, and Packet Sequence as DATA plus an explicit Valid Length indicating the number of meaningful application bytes in the final packet. Bytes after Valid Length are padding and are not delivered to the application.

The exact width and byte placement of Valid Length remain to be frozen with the packet-type/control layout, but the semantic rule is frozen: ordinary DATA always has the normal fixed-size interpretation, while only `DATA_END` carries a partial final payload length.

`DATA_END` participates in sequence numbering and acknowledgement exactly like DATA.

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

The Reserved byte MUST be transmitted as zero and ignored on receipt.

### ACK Base

`ACK Base` is the highest DATA/DATA_END packet sequence number such that that packet and every preceding packet in the current sequence space have been received correctly. `ACK Base = N` cumulatively acknowledges every packet through `N`.

### Receive Bitmap

The 32-bit Receive Bitmap selectively reports the next 32 sequence numbers after ACK Base:

```text
bit 0  -> ACK Base + 1
bit 1  -> ACK Base + 2
...
bit 31 -> ACK Base + 32
```

`1` means the complete DATA/DATA_END packet was received correctly. `0` means it is still missing.

### Receive Credit

`Receive Credit` is an unsigned 8-bit count of additional DATA/DATA_END packets that the receiver is prepared to accept on this stream. The unit is packets, not bytes.

`0` forbids introducing new sequence numbers until later credit is advertised. Retransmission of packets already outstanding does not consume new sequence-space credit. `255` means at least 255 additional packets may be accepted.

## ACK transmission timing

The initial GTS profile freezes the following rule:

- under normal in-order traffic, send an ACK after every two correctly received DATA/DATA_END packets;
- if only one packet is pending acknowledgement, send an ACK when the short delayed-ACK timer expires;
- send an ACK immediately when a gap is first observed;
- send an ACK immediately when a previously reported gap is filled and ACK Base can advance;
- send an ACK immediately when Receive Credit falls to zero or changes in a way needed to restart a stalled sender.

The exact delayed-ACK timer value remains an implementation parameter until timing constants are frozen; implementations must keep it short relative to the current measured round-trip time and may use a conservative fixed upper bound.

## Selective-repeat retransmission

When an ACK bitmap positively acknowledges a later packet while an earlier outstanding packet in the represented range remains zero, the sender treats the earlier packet as an explicitly identified hole and retransmits it immediately. GTS does not require duplicate-ACK counting or a separate fast-retransmit heuristic.

Packets positively acknowledged by ACK Base or bitmap bits are removed from retransmission state.

A retransmission timeout remains mandatory because the last outstanding packet, or an ACK itself, may be lost without creating a bitmap-visible hole.

## Adaptive retransmission timeout

The initial profile uses an adaptive retransmission timeout derived from measured round-trip time rather than a single fixed timeout for all paths.

Each stream/tunnel maintains a smoothed RTT estimate using ACKs for non-retransmitted packets. RTO is derived as a conservative multiple of the smoothed RTT and bounded by implementation-defined minimum and maximum values suitable for the network profile. Exact integer coefficients and bounds remain to be frozen with timing constants.

One retransmission timer per stream is sufficient for the initial implementation. On timeout, retransmit the oldest outstanding unacknowledged packet and restart the timer conservatively.

## Tunnel establishment and Stream 0

`CONNECT` establishes the tunnel and simultaneously opens **Stream 0** as the initial service stream. A separate STREAM_OPEN exchange is not required for the common one-stream case.

The initiator includes the receive Tunnel ID allocated for packets sent back toward it. The responder allocates and returns its own receive Tunnel ID in CONNECT_ACK. Thereafter each sender places the peer's receiver-local Tunnel ID in ordinary packets.

Additional streams are established with `STREAM_OPEN` / `STREAM_ACK`.

## Stream ID allocation

Stream IDs are 8 bits and use parity ownership to avoid simultaneous allocation collisions:

- the CONNECT initiator allocates **even** Stream IDs;
- the CONNECT responder allocates **odd** Stream IDs;
- Stream 0 is created by CONNECT and belongs to the initiator side of the allocation convention.

An endpoint MUST NOT allocate a Stream ID owned by the peer's parity. Reuse after closure requires the previous incarnation to be fully retired according to the final stale-packet rules.

## Graceful stream close

Stream closure is directional. `STREAM_CLOSE` means the sender will transmit no further DATA/DATA_END packets in that direction on the stream. The opposite direction may remain open and continue sending.

A stream is fully closed only when both directions have been gracefully closed and all required acknowledgement/close confirmation state has completed.

## RESET

`RESET` is the fatal/abnormal termination mechanism. It destroys the tunnel and all streams immediately rather than waiting for outstanding DATA or graceful close state.

The exact RESET wire layout and whether the previously discussed Reset ID/capability remains mandatory are still open; the semantic distinction is frozen: graceful close is directional and orderly, RESET is immediate tunnel-wide failure termination.

## DATA/ACK separation

ACK is always a distinct GTS packet type. DATA and DATA_END MUST NOT piggyback ACK state in the initial profile. Bidirectional streams therefore maintain independent DATA sequence spaces and independent ACK traffic in each direction.

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

CRC input is the canonical GDP pseudo-header followed by the complete GTS header and payload. The CRC trailer itself is excluded.

```text
GDP Version
GDP Type = 0x2 (GTS)
GDP Size Class
effective 64-bit Source Address
effective 64-bit Destination Address
complete GTS header
complete GTS payload
```

For Global GDP, effective addresses are the transmitted 64-bit addresses. For Local GDP, the local IDs are expanded to their canonical 64-bit endpoint identities before CRC calculation. Hop Limit, Local/Global representation, reserved bits, and mutable forwarding state are excluded.

A failed CRC causes the packet to be discarded and treated as not received.

## Service Selector field

Service selection remains setup-only and uses the accepted variable-width selector model. Ordinary DATA/ACK packets do not carry Service Selectors.

## Remaining open wire work

The behavioral model above is frozen. Remaining initial-profile wire work is limited to:

- exact numeric GTS packet-type assignments;
- exact CONNECT / CONNECT_ACK layout;
- exact STREAM_OPEN / STREAM_ACK layout;
- exact DATA_END Valid Length width/placement;
- exact STREAM_CLOSE / close-confirmation layout;
- exact RESET layout and Reset-ID decision;
- delayed-ACK/RTO numeric timing constants;
- stale-packet/Stream-ID reuse guard rules;
- padding rules and golden packet vectors.

Unknown/undefined GTS packet types are discarded.
