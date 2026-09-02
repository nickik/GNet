# GNet Transport and Session Protocol (GTS)

Status: **OPEN wire format**

GTS is the endpoint protocol family above GDP. It must support unreliable messages, reliable ordered delivery, multiplexed sessions, and reserved real-time flows without adding state to ordinary GDP forwarding.

A complete specification must define connection establishment, collision handling, local and global identifiers, ports, segmentation, sequence space, acknowledgments, receive-window units, retransmission timers, duplicate suppression, reset/release, keepalive, checksum, cryptographic binding, QoS/reservation requests, and path-change behavior.

The historic CONNECT layouts are retained in [packets/transport.md](../packets/transport.md), including their unresolved bit-count problems. They are input to the design, not yet normative encodings.
