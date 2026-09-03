---
id: specification-status
title: "Specification Status"
aliases: ["GNet status"]
type: meta
status: mixed
tags: ["gnet","gnet/meta","gnet/status/mixed"]
parent: "[[GNet Home]]"
related: ["[[Open Questions]]","[[GNet Architecture Overview]]"]
updated: 2026-09-02
---
# Specification status

> [!info] Knowledge graph
> **Up:** [[GNet Home]] · **Related:** [[Open Questions]] · [[GNet Architecture Overview]]


This repository is **GNet Draft 0.1**. It is sufficient to discuss interoperable components, but not yet sufficient to build independently interoperable implementations.

## FROZEN constraints

- Direct Link Protocol (DLP) is L2; GDP is the minimal routed L3 datagram; GNet session and transport functions are above GDP.
- Every transmitted flit is exactly 32 bits: a 4-bit hop-local VCID followed by 28 carried bits. The VCID is inside the flit, not sideband.
- VCIDs identify active bounded DLP segments, are scoped to a link and direction plus any medium-supplied transmitter/channel context, and are replaced at every router.
- DLP Segment Class values `00`, `01`, and `10` bound meaningful payloads to 64, 256, and 1,024 octets respectively; `11` is reserved. Payload Length gives the exact octet count within the class limit.
- GDP contains only version, type, hop limit, QoS, source address, and destination address.
- GDP has no payload length, checksum, CRC, hash, integrity flag/trailer, session or flow identifier, reliability state, fragmentation state, or options.
- GNet addresses are 64-bit and hierarchical. A customer receives at least eight device/suffix bits. Endpoints remain globally reachable; NAT is not part of the architecture.
- Links are point-to-point, physically identified, or centrally arbitrated. Core operation never depends on Ethernet-style MAC learning or a shared collision domain.
- Hop-local integrity belongs to DLP; end-to-end integrity belongs to GTS or another protocol carried by GDP. A DLP integrity trailer is not part of the GDP packet.
- A DLP logical line has one directly attached endpoint and carries no network addresses or session state.
- Retransmission, session tracking, encryption, and application identity are endpoint concerns, not router forwarding concerns.
- Voice is a reserved packet flow. Conversion to legacy circuit telephony occurs at gateways.
- Usage accounting is outside the forwarding fast path.

## ACCEPTED direction

- GNET-L is a roughly 3 Mb/s local star over four twisted pairs with request/grant arbitration.
- GNET-A is centrally polled residential access for roughly 256 premises, targeting 10 Mb/s and reserved real-time capacity.
- GNET-P is synchronous point-to-point infrastructure at 10, 25, and 50 Mb/s, initially over coax and later fiber.
- A generic service solicitation identifies routers, directories, terminal servers, boot servers, and future service classes.
- The only unavoidable local fanout is an initial solicitation. Advertisements and subsequent configuration are returned directly.
- Bootstrap order is router discovery, address configuration, directory discovery, then named service selection.
- Native GNet switches may be QDX internally. Small systems may integrate the same architecture on one PCB; large systems may use QDX backplanes and cards.
- GTS is tunnel-first: destructive close/reset/rebind operations require a Reset ID, while ordinary data omits it; independently configured streams are multiplexed inside a tunnel.
- Logical service names may resolve to multiple providers, including providers outside the local subnet.
- GSC signaling is separated from direct endpoint-to-endpoint media/data flow.
- Wire formats are presented and packaged as complete 32-bit flits containing 4-bit VCID plus 28 carried bits; each physical medium may serialize those bits differently.
- Several active VCIDs may be interleaved on a common link; bounded segments ensure eventual VCID and forwarding-resource release.

## DRAFT items

- All numeric protocol, service, and message allocations.
- The 20-octet GDP encoding and QoS semantics. Its six-flit DLP packing follows mechanically from the frozen 4+28 flit format.
- DLP padding and integrity-trailer rules.
- The 8-bit Protocol, 2-bit Link Class, 2-bit Segment Class, and 16-bit Payload Length header encodings; integrity algorithm and polynomial.
- VCID value allocation, allocation/release signaling, timeout and error recovery, persistent Link Flow ID binding, and interleaving requirements.
- Discovery and address-configuration payload layouts.
- Transport packet layouts and algorithms.
- Tunnel/Reset/Stream identifier widths, reserved stream values, and the final port-versus-service-selector design.

## Interpretation rule

When documents conflict, the most recently accepted decision wins, followed by FROZEN status, then ACCEPTED, then DRAFT. Open questions must not be treated as implicit requirements.
