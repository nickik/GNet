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

## Tunnel and stream model

GTS first establishes a tunnel. A separate Reset ID is the capability required to close, reset, or rebind it; normal data does not repeat that capability. Multiple streams share the tunnel and negotiate reliability, ordering/sequencing, byte-versus-message delivery, encryption, and compression independently. Service selection belongs in CONNECT or STREAM_OPEN rather than ordinary data.

The current working proposal uses a 64-bit Tunnel ID, 64-bit Reset ID, 16-bit Stream IDs, stream 0 for control, and stream 1 as default data. Only the tunnel-first model and reset-authority behavior are accepted; the widths and reserved stream numbers remain DRAFT.

The architecture should support at least three service modes: unreliable datagram, reliable ordered stream/message, and reserved real-time flow. They may share a common session-control header, but routers must not need transport state for ordinary forwarding.

## Real-time flows

Voice, interactive media, and other bounded-delay traffic remain GDP packets. Endpoints request resources through GNet Session Control. Routers perform admission control and schedule the admitted traffic using GDP QoS plus local reservation state. Legacy telephone circuits terminate only at district or edge gateways.

## Historical transport draft

The project previously selected detailed CONNECT and CONNECT_ACK field lists. Their totals are not octet-aligned and CONNECT contains two receive-window fields. They are preserved exactly in [packets/transport.md](../../packets/transport.md) as requirements evidence, but are not yet an interoperable encoding.
