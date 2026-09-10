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

Status: **FROZEN ordinary DATA and ACK layouts; OPEN connection/control packing**

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

The Packet Sequence field is 32 bits. Sequence numbers identify DATA packets, not byte offsets, and are interpreted modulo `2^32`. Sequence spaces are independent for each Stream ID within a tunnel.

## ACK packet

The ACK packet format is frozen as:

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

Logical layout:

```text
+-------------------------------+
| Version 4 | Type = ACK 4      | 1 byte
+-------------------------------+
| Receiver Tunnel ID            | 4 bytes
+-------------------------------+
| Stream ID                     | 1 byte
+-------------------------------+
| ACK Base                      | 4 bytes
+-------------------------------+
| Receive Bitmap                | 4 bytes
+-------------------------------+
| Receive Credit                | 1 byte
+-------------------------------+
| Reserved                      | 1 byte
+-------------------------------+
| CRC-32-GNET                   | 4 bytes
+-------------------------------+
```

The Reserved byte MUST be transmitted as zero and ignored on receipt in this version.

ACK is normally carried in the 32-byte GDP Size Class. Unused bytes in the enclosing fixed-size GDP payload are padding/reserved according to the final GTS padding rule.

### ACK Base

`ACK Base` is the highest DATA packet sequence number such that that packet and every preceding packet in the current receive sequence space have been received correctly.

Thus `ACK Base = N` cumulatively acknowledges every packet through `N`.

### Receive Bitmap

The 32-bit Receive Bitmap selectively reports receipt of the next 32 packet sequence numbers after ACK Base:

```text
bit 0  -> ACK Base + 1
bit 1  -> ACK Base + 2
...
bit 31 -> ACK Base + 32
```

A bit value of `1` means the corresponding complete DATA packet has been received correctly and retained/accepted by the receiver. A bit value of `0` means it has not been received correctly and is still considered missing.

The sender retransmits missing packets as required by the GTS retransmission rules; packets positively acknowledged by ACK Base or a set bitmap bit need not be retransmitted.

### Receive Credit

`Receive Credit` is an unsigned 8-bit count of additional DATA packets that the receiver is currently prepared to accept for this stream beyond the packets already represented as received/outstanding state.

The unit is packets, not bytes. Because each stream uses packet-sequence semantics, this allows simple receiver buffer accounting.

`0` means the sender must not introduce new DATA packets for that stream until later ACK state advertises positive credit. Retransmission of already-outstanding packets remains governed by the final retransmission/credit rule and must not be interpreted as creation of new sequence state.

`255` means at least 255 additional DATA packets may be accepted; no larger value is represented in the initial format.

### Selective-repeat behavior

GTS acknowledgements operate on whole DATA packets, never byte ranges. The receiver may retain correctly received packets beyond a gap and identify them in the bitmap. Application delivery remains ordered: later packets are not delivered ahead of a missing earlier packet unless a future stream profile explicitly changes this rule.

Example:

```text
Received: 100 101 [102 missing] 103 104 [105 missing] 106

ACK Base       = 101
Bitmap bit 0   = 0   ; packet 102 missing
Bitmap bit 1   = 1   ; packet 103 received
Bitmap bit 2   = 1   ; packet 104 received
Bitmap bit 3   = 0   ; packet 105 missing
Bitmap bit 4   = 1   ; packet 106 received
```

If packet 102 later arrives, ACK Base may advance through all contiguous packets already present, stopping at the next hole.

## DATA/ACK separation

ACK is a distinct GTS packet type. DATA MUST NOT piggyback an ACK block in the initial profile. Bidirectional streams therefore use independent DATA packets and independent ACK packets in each direction.

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

The exact numeric packet-type registry and detailed connection/control packet layouts remain under review. At minimum the initial profile needs CONNECT, CONNECT_ACK, STREAM_OPEN, STREAM_ACK, DATA, ACK, STREAM_CLOSE, tunnel close, and RESET semantics.

Unknown/undefined GTS packet types in the final registry are discarded. No implementation may reinterpret an undefined type as DATA or another known packet form.

The next wire-format decisions are the CONNECT/STREAM_OPEN/close/reset control packets, ACK transmission timing, retransmission timing, and final padding/golden-vector rules.
