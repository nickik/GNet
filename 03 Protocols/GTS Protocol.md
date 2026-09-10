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

Status: **OPEN exact packet layouts; ACCEPTED initial tunnel/stream, service-selector, and integrity semantics**

GTS is the endpoint transport/session protocol above GDP. The initial traditional packet-routed profile is deliberately narrow: reliable ordered streams over ordinarily routed GDP packets. Unreliable, multicast/group, flow-ID routed, and congestion-control extensions are deferred.

## Tunnel and stream identity

The initial profile freezes:

- **Tunnel ID: 32 bits, receiver-local.** Each endpoint allocates the Tunnel ID that its peer uses when sending packets to it. A Tunnel ID need only be unique among active tunnels at the receiving endpoint; it is not globally unique.
- **Stream ID: 8 bits, scoped to one tunnel.** Stream identity is therefore compact and independent of GDP addresses.

A CONNECT exchange must establish the receiving Tunnel ID for each direction before ordinary DATA can flow. Ordinary packets carry only the destination/receiver-local Tunnel ID, not both endpoints' Tunnel IDs.

Stream-allocation parity, reserved Stream IDs, and the exact CONNECT/STREAM_OPEN state machine remain open pending the packet-layout review.

A separate Reset ID/capability remains part of the design for destructive reset/close/rebind operations, but its final width and exact state-machine use remain open.

## End-to-end integrity

GDP deliberately carries no checksum or CRC. GTS therefore provides mandatory end-to-end integrity for every ordinary GTS packet.

CRC width is fixed by the enclosing GDP Size Class and is not negotiated or explicitly encoded:

- the dedicated zero-byte and 3-byte tiny GDP classes use CRC-8 when a future compact GTS form permits their use;
- every larger GDP Size Class uses CRC-32.

The ordinary initial GTS format cannot fit in the 0-byte or 3-byte GDP classes once the 32-bit Tunnel ID and 8-bit Stream ID are present. Traditional GTS therefore uses the larger classes and CRC-32. The tiny CRC-8 rule is retained for a future compact/tiny GTS encoding rather than forcing CRC-32 overhead onto such a form.

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

The GTS CRC is calculated over a canonical end-to-end context followed by the complete GTS header and payload. For the traditional routed profile, the canonical GDP contribution is:

```text
GDP Version / protocol context
GDP Size Class
Effective Source Address      64 bits
Effective Destination Address 64 bits
[GDP Type, if retained in the final GDP header]
GTS header
GTS payload
```

The **effective** source and destination are the canonical 64-bit endpoint identities. If a local/compact GDP representation abbreviates an endpoint address on a particular link, that address is expanded to its effective 64-bit identity before CRC calculation. Thus changing between compact and full wire representations at a router does not change the end-to-end GTS CRC context.

The local/global address-mode bits themselves are representation details and are not CRC input. Hop Limit is excluded because routers legitimately decrement it. Reserved bits and other mutable hop-local representation state are excluded.

If the current GDP `Type` field remains as a next-protocol discriminator, its value MUST be included in the pseudo-header. If that field is removed from the initial GDP format, GTS uses a fixed protocol-domain constant instead. This one point remains tied to the pending GDP Type decision.

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

The exact restricted character alphabet/packing for textual selectors remains open; the direction under discussion is a compact uppercase-letter/`-` grammar rather than unrestricted ASCII.

The selector is carried only during service setup. Ordinary DATA packets do not repeat it.

Sparse long selectors can make exhaustive remote service scanning impractical, but this is not cryptographic protection. Predictable names remain guessable and passive observers can learn selectors from setup traffic.

Service enumeration is optional and belongs to higher-level discovery/directory facilities.

## Explicitly deferred from the initial profile

The following are not required to finish the traditional packet-routed GTS baseline:

- flow-ID or virtual-circuit routed optimization;
- multicast/group streams;
- end-to-end congestion-control algorithm;
- unreliable stream profiles;
- cryptographic authentication/encryption;
- dynamic route behavior.

The remaining initial-profile work is the exact GTS packet-type registry, CONNECT/STREAM_OPEN state machines, DATA and ACK layouts, packet sequence and acknowledgement rules, receive-credit encoding, retransmission timing, graceful close/reset behavior, and golden packet vectors.
