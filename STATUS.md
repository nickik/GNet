# Specification status

This repository is **GNet Draft 0.1**. It is sufficient to discuss interoperable components, but not yet sufficient to build independently interoperable implementations.

## FROZEN constraints

- Direct Link Protocol (DLP) is L2; GDP is the minimal routed L3 datagram; GNet session and transport functions are above GDP.
- GDP contains only version, type, hop limit, QoS, source address, and destination address.
- GDP has no payload length, integrity field, session or flow identifier, reliability state, fragmentation state, or options.
- GNet addresses are 64-bit and hierarchical. A customer receives at least eight device/suffix bits. Endpoints remain globally reachable; NAT is not part of the architecture.
- Links are point-to-point, physically identified, or centrally arbitrated. Core operation never depends on Ethernet-style MAC learning or a shared collision domain.
- DLP uses an 8-bit CRC. Stronger end-to-end integrity belongs above GDP.
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
- Wire formats are presented and packaged as 32-bit logical flits; each physical medium may serialize them differently.

## DRAFT items

- All numeric protocol, service, and message allocations.
- The five-flit/20-octet GDP encoding.
- The common 32-bit flit ordering and DLP padding/CRC packaging rules.
- DLP frame field widths, frame size, and CRC polynomial.
- Discovery and address-configuration payload layouts.
- Transport packet layouts and algorithms.
- Tunnel/Reset/Stream identifier widths, reserved stream values, and the final port-versus-service-selector design.

## Interpretation rule

When documents conflict, the most recently accepted decision wins, followed by FROZEN status, then ACCEPTED, then DRAFT. Open questions must not be treated as implicit requirements.
