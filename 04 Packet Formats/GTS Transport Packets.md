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

Status: **OPEN exact DATA/ACK/control packing; identifier and integrity widths accepted**

The initial traditional packet-routed GTS profile uses:

- 32-bit receiver-local Tunnel IDs;
- 8-bit Stream IDs scoped to one tunnel;
- reliable ordered streams;
- packet-by-packet GDP routing;
- mandatory end-to-end CRC.

Flow-ID routing, multicast/group transport, unreliable streams, and transport congestion control are outside this initial profile.

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
+-----------------------------------------------+
| canonical GDP pseudo-header                   |
|  effective source address                     |
|  effective destination address                |
|  GDP Size Class                               |
|  GDP protocol context / Type if retained      |
+-----------------------------------------------+
| complete GTS header                           |
+-----------------------------------------------+
| complete GTS payload                          |
+-----------------------------------------------+
             -> CRC-8 or CRC-32
```

For the traditional routed profile, effective source and destination are canonical 64-bit GDP endpoint identities. A compact/local GDP representation must expand its abbreviated addresses to those same effective endpoint identities before CRC calculation. Thus local/global wire representation may change at a router without changing GTS end-to-end integrity.

Hop Limit, local/global representation bits, reserved bits, and other mutable forwarding/representation fields are excluded from the pseudo-header. If GDP retains its current next-protocol `Type` field, that Type is included; if it is removed, a fixed GTS protocol-domain constant replaces it.

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

## Packet layouts still to freeze

The exact numeric packet-type registry and detailed packet layouts remain under review. At minimum the initial profile needs CONNECT, CONNECT_ACK, STREAM_OPEN, STREAM_ACK/ACCEPT, DATA, ACK, STREAM_CLOSE, tunnel close, and RESET semantics.

Unknown/undefined GTS packet types in the final registry are discarded. No implementation may reinterpret an undefined type as DATA or another known packet form.

The next wire-format decision is the exact DATA and ACK layout, including sequence-number width, ACK bitmap/credit representation, and whether any DATA flags are required.
