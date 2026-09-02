# Transport, sessions, and reserved flows

Status: **FROZEN placement; OPEN wire protocol**

GDP provides best-effort routed datagrams. Endpoints add the following functions as needed:

- source and destination ports;
- session/tunnel identity and local handles;
- sequence and acknowledgment numbers;
- receive windows and flow control;
- retransmission and ordered delivery;
- fragmentation and reassembly when payloads exceed a link path limit;
- end-to-end integrity and optional encryption;
- reserved-flow setup and release.

The architecture should support at least three service modes: unreliable datagram, reliable ordered stream/message, and reserved real-time flow. They may share a common session-control header, but routers must not need transport state for ordinary forwarding.

## Real-time flows

Voice, interactive media, and other bounded-delay traffic remain GDP packets. Endpoints request resources through GNet Session Control. Routers perform admission control and schedule the admitted traffic using GDP QoS plus local reservation state. Legacy telephone circuits terminate only at district or edge gateways.

## Historical transport draft

The project previously selected detailed CONNECT and CONNECT_ACK field lists. Their totals are not octet-aligned and CONNECT contains two receive-window fields. They are preserved exactly in [packets/transport.md](../../packets/transport.md) as requirements evidence, but are not yet an interoperable encoding.
