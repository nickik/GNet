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
updated: 2026-09-02
---
# GNet Transport and Session Protocol (GTS)

> [!info] Knowledge graph
> **Up:** [[Protocols MOC]] · **Related:** [[GTS Transport Packets]] · [[Transport and Flows]] · [[ADR-0005 Tunnels and Streams]] · [[ADR-0009 No GDP Integrity Field]]


Status: **OPEN wire format**

GTS is the endpoint protocol family above GDP. It must support unreliable messages, reliable ordered delivery, multiplexed sessions, and reserved real-time flows without adding state to ordinary GDP forwarding.

GTS uses a tunnel-first model. A Tunnel ID identifies the association; a Reset ID authorizes close, reset, and rebind but is omitted from ordinary data. Streams within the tunnel may independently select reliable/unreliable, ordered/sequenced, message/byte, encryption, and compression behavior.

A complete specification must define connection establishment, collision handling, local and global identifiers, numeric ports and/or setup-only service selectors, stream open/accept, segmentation, sequence space, acknowledgments, receive-window units, retransmission timers, duplicate suppression, reset/release/rebind, keepalive, end-to-end checksum, cryptographic binding, QoS/reservation requests, and path-change behavior. GDP supplies no checksum or other integrity field, so GTS must define the end-to-end behavior required by each stream profile.

The historic CONNECT layouts are retained in [[GTS Transport Packets]], including their unresolved bit-count problems. They are input to the design, not yet normative encodings.
