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

Status: **FROZEN ordinary DATA layout and core identifier widths; OPEN ACK/control packing**

The initial traditional packet-routed GTS profile uses:

- 32-bit receiver-local Tunnel IDs;
- 8-bit Stream IDs scoped to one tunnel;
- 32-bit packet sequence numbers scoped to one stream;
- reliable ordered streams;
- separate DATA and ACK packet types;
- packet-by-packet GDP routing;
- mandatory end-to-end CRC.

Flow-ID routing, multicast/group transport, unreliable streams, and transport congestion control are outside this initial profile.

## Ordinary DATA packet

The ordinary DATA packet has a fixed 10-byte GTS header followed by application payload and a 4-byte CRC-32 trailer:

```text
+-------------------------------+
| Version 4 | Type 4            | 1 byte
+-------------------------------+
| Receiver Tunnel ID            | 4 bytes
+-------------------------------+
| Stream ID                     | 1 byte
+-------------------------------+
| Packet Sequence               | 4 bytes
+-------------------------------+
| Application payload           | variable within GDP Size Class
+-------------------------------+
| CRC-32-GNET                   | 4 bytes
+-------------------------------+
```

Fixed overhead is therefore 14 bytes:

```text
1  Version/Type
4  Tunnel ID
1  Stream ID
4  Sequence
4  CRC-32
--------------
14 bytes
```

DATA does not carry acknowledgement state, receive credit, service selector, Reset ID, source Tunnel ID, QoS, flags, or a payload-length field.

The Packet Sequence field is 32 bits. Sequence numbers are interpreted modulo `2^32` and are independent for each Stream ID within a tunnel.

## DATA/ACK separation

ACK is a distinct GTS packet type. DATA MUST NOT piggyback an ACK block in the initial profile. Bidirectional streams therefore use independent DATA packets and independent ACK packets in each direction.

The exact ACK layout remains OPEN. The working reliability direction remains selective acknowledgement using an ACK base plus a fixed bitmap and receive-credit indication.

## Mandatory integrity trailer

Every GTS packet carries an end-to-end CRC trailer. CRC width is determined by the enclosing GDP Size Class and is not negotiated or explicitly encoded.

| GDP Size Class | GTS CRC |
|---|---:|
| dedicated 0-byte tiny class | CRC-8 only for a future compact GTS form |
| dedicated 3-byte tiny class | CRC-8 only for a future compact GTS form |
| every larger class | CRC-32 |

The ordinary 32-bit-Tunnel/8-bit-Stream GTS header cannot fit in the 0-byte or 3-byte GDP classes. Traditional GTS therefore uses larger GDP classes and CRC-32.

CRC parameters are frozen as:

```text
CRC-8-GNET
  width       8
  polynomial  0x07
  init        0x00
  refin       false
  refout      false
  xorout      0x00
  check       "123456789" -> 0xF4

CRC-32-GNET
  width       32
  polynomial  0x04C11DB7
  init        0xFFFFFFFF
  refin       false
  refout      false
  xorout      0xFFFFFFFF
  byte order  most-significant byte first
  check       "123456789" -> 0xFC891918
```

The CRC input is the canonical GDP pseudo-header followed by the complete GTS header and payload. The CRC trailer itself is not included in the calculation.

```text
GDP Version
GDP Type = 0x2 (GTS)
GDP Size Class
effective 64-bit Source Address
effective 64-bit Destination Address
complete GTS header
complete GTS payload
        -> CRC-32-GNET for ordinary GTS
```

For Global GDP, the effective addresses are the transmitted 64-bit addresses. For Local GDP, both transmitted addresses are 8-bit local IDs and are expanded using the known local prefix/context to their canonical 64-bit endpoint addresses before CRC calculation.

Hop Limit, the Local/Global representation bit, reserved bits, and other mutable representation fields are excluded.

A failed CRC causes the packet to be discarded and treated as not received. Corrupted DATA MUST NOT be positively acknowledged.

## Service Selector field

When setup selects a logical service, it carries a 2-bit selector class followed by a class-specific selector field:

| Class | Selector field | Current intent |
|---:|---:|---|
| `00` | 8 bits | numeric registered service code |
| `01` | short fixed field | compact textual service selector; exact alphabet/packing open |
| `10` | 128 bits | long/private textual selector |
| `11` | reserved | future expansion |

The selector is setup-only. Once a service has been accepted and bound to tunnel/stream state, DATA packets use only Tunnel ID and Stream ID.

## Packet types still to freeze

The exact numeric packet-type registry and detailed control packet layouts remain under review. At minimum the initial profile needs CONNECT, CONNECT_ACK, STREAM_OPEN, STREAM_ACK, DATA, ACK, STREAM_CLOSE, tunnel close, and RESET semantics.

Unknown/undefined GTS packet types in the final registry are discarded. No implementation may reinterpret an undefined type as DATA or another known packet form.

The next wire-format decision is the exact ACK layout and then the CONNECT/STREAM_OPEN/close/reset control packets.
