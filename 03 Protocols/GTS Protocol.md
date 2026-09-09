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
updated: 2026-09-09
---
# GNet Transport and Session Protocol (GTS)

> [!info] Knowledge graph
> **Up:** [[Protocols MOC]] · **Related:** [[GTS Transport Packets]] · [[Transport and Flows]] · [[ADR-0005 Tunnels and Streams]] · [[ADR-0009 No GDP Integrity Field]]


Status: **OPEN wire format; ACCEPTED tunnel/stream and service-selector semantics**

GTS is the endpoint protocol family above GDP. It must support unreliable messages, reliable ordered delivery, multiplexed sessions, and reserved real-time flows without adding state to ordinary GDP forwarding.

GTS uses a tunnel-first model. A Tunnel ID identifies the association; a Reset ID authorizes close, reset, and rebind but is omitted from ordinary data. Streams within the tunnel may independently select reliable/unreliable, ordered/sequenced, message/byte, encryption, and compression behavior.

## Service selection

GTS does not require TCP-style 16-bit source and destination ports. Transport identity and service identity are separate:

- Tunnel ID identifies an established transport association;
- Stream ID identifies one flow within a tunnel;
- Service Selector identifies the logical service requested during setup.

The Service Selector is variable-width. It consists logically of a 4-bit Service Size Class followed by a Service ID:

| Size class | Service ID width |
|---:|---:|
| 0 | 8 bits |
| 1 | 16 bits |
| 2 | 32 bits |
| 3 | 64 bits |
| 4 | 128 bits |
| 5-15 | reserved |

The selector is carried only when selecting/opening a service, in CONNECT and/or STREAM_OPEN depending on the final state machine. Ordinary DATA packets do not repeat it.

Class 0 provides a compact public/common service namespace. Wider classes allow much larger namespaces and may be allocated sparsely. Sparse 64-bit or 128-bit selectors can make exhaustive remote service scanning impractical without introducing cryptography. This property depends on allocation: sequential or otherwise predictable values remain guessable regardless of field width.

The selector mechanism itself provides no secrecy or authentication. A passive observer can learn a selector that appears in setup traffic. Cryptographic protection is outside the current baseline.

Service enumeration is optional and belongs to higher-level discovery/directory facilities; GTS does not require a node to advertise every selector it accepts.

A complete specification must still define connection establishment, collision handling, local and global identifiers, exact selector placement, stream open/accept, segmentation, sequence space, acknowledgments, receive-window units, retransmission timers, duplicate suppression, reset/release/rebind, keepalive, end-to-end checksum, optional cryptographic binding, QoS/reservation requests, and path-change behavior. GDP supplies no checksum or other integrity field, so GTS must define the end-to-end behavior required by each stream profile.

The historic CONNECT layouts are retained in [[GTS Transport Packets]], including their unresolved bit-count problems. They are input to the design, not yet normative encodings.
