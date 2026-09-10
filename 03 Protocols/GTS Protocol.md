---
id: gts-protocol
title: "GTS Protocol"
aliases: ["GTS","GNet Transport and Session Protocol"]
type: protocol
status: open
layers: ["L4","L5"]
tags: ["gnet","gnet/protocol","gnet/status/open","gnet/layer/l4","gnet/layer/l5"]
parent: "[[Protocols MOC]]"
related: ["[[GTS Transport Packets]]","[[Transport and Flows]]","[[ADR-0005 Tunnels and Streams]]","[[ADR-0009 No GDP Integrity Field]]"]
updated: 2026-09-10
---
# GNet Transport and Session Protocol (GTS)

> [!info] Knowledge graph
> **Up:** [[Protocols MOC]] · **Related:** [[GTS Transport Packets]] · [[Transport and Flows]] · [[ADR-0005 Tunnels and Streams]] · [[ADR-0009 No GDP Integrity Field]]

Status: **OPEN control/ACK layouts; ACCEPTED initial DATA, tunnel/stream, sequence, service-selector, and integrity semantics**

GTS is the endpoint transport/session protocol above GDP. The initial traditional packet-routed profile is deliberately narrow: reliable ordered streams over ordinarily routed GDP packets. Unreliable, multicast/group, flow-ID routed, and congestion-control extensions are deferred.

## Tunnel and stream identity

The initial profile freezes:

- **Tunnel ID: 32 bits, receiver-local.** Each endpoint allocates the Tunnel ID that its peer uses when sending packets to it.
- **Stream ID: 8 bits, scoped to one tunnel.**
- **Packet Sequence: 32 bits, scoped to one stream.** Sequence arithmetic is modulo `2^32`.

A CONNECT exchange must establish the receiving Tunnel ID for each direction before ordinary DATA can flow. Ordinary packets carry only the destination/receiver-local Tunnel ID, not both endpoints' Tunnel IDs.

Stream-allocation parity, reserved Stream IDs, and the exact CONNECT/STREAM_OPEN state machine remain open pending the control-packet layout review.

A separate Reset ID/capability remains part of the design for destructive reset/close operations, but its final width and exact state-machine use remain open.

## DATA and ACK separation

The initial GTS profile uses **separate DATA and ACK packet types**. DATA never piggybacks an ACK block. This keeps the common data path fixed and small.

The accepted ordinary DATA fixed header/trailer overhead is 14 bytes:

```text
1 byte   Version/Type
4 bytes  Receiver-local Tunnel ID
1 byte   Stream ID
4 bytes  Packet Sequence
4 bytes  CRC-32
-------------------------------
14 bytes fixed overhead
```

The payload occupies the remaining GTS bytes supplied by the enclosing GDP Size Class. There is no GTS payload-length field in DATA; the GDP Size Class determines the enclosing packet budget.

## End-to-end integrity

GDP deliberately carries no checksum or CRC. GTS therefore provides mandatory end-to-end integrity for every ordinary GTS packet.

CRC width is fixed by the enclosing GDP Size Class and is not negotiated or explicitly encoded:

- the dedicated zero-byte and 3-byte tiny GDP classes use CRC-8 when a future compact GTS form permits their use;
- every larger GDP Size Class uses CRC-32.

The ordinary initial GTS format cannot fit in the 0-byte or 3-byte GDP classes once the 32-bit Tunnel ID and 8-bit Stream ID are present. Traditional GTS therefore uses the larger classes and CRC-32.

### CRC-8-GNET

- width: 8
- polynomial: `0x07` (`x^8 + x^2 + x + 1`)
- initial register: `0x00`
- input/output reflection: no
- final XOR: `0x00`
- check value for ASCII `123456789`: `0xF4`

### CRC-32-GNET

- width: 32
- polynomial: `0x04C11DB7`
- initial register: `0xFFFFFFFF`
- input/output reflection: no
- final XOR: `0xFFFFFFFF`
- transmitted CRC byte order: most-significant byte first
- check value for ASCII `123456789`: `0xFC891918`

These choices deliberately favor a simple MSB-first shift-register implementation appropriate to early hardware.

### GDP pseudo-header binding

The GTS CRC is calculated over a canonical GDP context followed by the complete GTS header and payload. The initial pseudo-header is:

```text
GDP Version                 4 bits, canonicalized
GDP Type                    4 bits; GTS = 0x2
GDP Size Class              4 bits
Effective Source Address   64 bits
Effective Destination      64 bits
GTS header
GTS payload
```

For Global GDP, the effective addresses are the transmitted 64-bit addresses. For Local GDP, the 8-bit local Source and Destination IDs are expanded using the known local prefix/context to their canonical 64-bit GDP addresses before CRC calculation.

The Local/Global representation bit itself is not CRC input. Hop Limit is excluded because routers legitimately decrement it. Reserved bits and mutable hop-local representation state are excluded.

A packet that fails GTS CRC validation is discarded and treated as not received. No positive acknowledgement may be generated from failed DATA.

CRC provides accidental-error detection, not authentication or secrecy.

## Service selection

GTS does not use TCP-style fixed 16-bit source and destination ports. Transport identity and service identity are separate:

- Tunnel ID identifies an established transport association;
- Stream ID identifies one flow within a tunnel;
- Service Selector identifies the logical service requested during setup.

The Service Selector begins with a 2-bit Size Class:

| Size class | Selector field | Representation |
|---:|---:|---|
| 0 | 8 bits | numeric registered service code |
| 1 | 32 bits | short fixed-width textual service selector |
| 2 | 128 bits | long fixed-width textual/private selector |
| 3 | reserved | future expansion |

The exact restricted character alphabet/packing for textual selectors remains open.

The selector is carried only during service setup. Ordinary DATA packets do not repeat it.

Service enumeration is optional and belongs to higher-level discovery/directory facilities.

## Explicitly deferred from the initial profile

The following are not required to finish the traditional packet-routed GTS baseline:

- flow-ID or virtual-circuit routed optimization;
- multicast/group streams;
- end-to-end congestion-control algorithm;
- unreliable stream profiles;
- cryptographic authentication/encryption;
- dynamic route behavior.

The remaining initial-profile work is the exact GTS packet-type registry, CONNECT/STREAM_OPEN state machines, ACK layout and bitmap/credit rules, retransmission timing, graceful close/reset behavior, and golden packet vectors.
