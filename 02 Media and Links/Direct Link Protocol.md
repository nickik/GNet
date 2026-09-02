---
id: direct-link-protocol
title: "Direct Link Protocol"
aliases: ["DLP","GNET-LINK"]
type: protocol
status: mixed
layers: ["L2"]
tags: ["gnet","gnet/protocol","gnet/status/mixed","gnet/layer/l2"]
parent: "[[Media and Links MOC]]"
related: ["[[32-bit Flit Format]]","[[GDP Protocol]]","[[ADR-0004 Direct Point-to-Point Link]]"]
updated: 2026-09-02
---
# Direct Link Protocol (DLP / GNET-LINK)

> [!info] Knowledge graph
> **Up:** [[Media and Links MOC]] · **Related:** [[32-bit Flit Format]] · [[GDP Protocol]] · [[ADR-0004 Direct Point-to-Point Link]]


Status: **FROZEN role; DRAFT encoding**

DLP is the common L2 envelope used by GNET-L, GNET-A, and GNET-P. It identifies the carried protocol, delimits a payload, and detects link corruption. The medium-specific layer supplies ingress/egress port or channel identity.

## Draft 32-bit flit frame

```text
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |    Protocol   |  Link Class   |    Payload Length (octets)    | Flit 0
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Payload octets 0 .. 3                      | Flit 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Payload octets 4 .. 7                      | Flit 2
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                              ...                              |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | Last payload octets; unused trailing octets MUST be zero      | Flit P
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |     CRC-8     |                Reserved = 0                   | Flit P+1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

**Protocol** selects GDP, link-local GCTL, or a registered foreign protocol. **Link Class** is a medium-local scheduling class; zero is ordinary best effort. **Payload Length** is the exact number of meaningful payload octets and does not include zero padding or the CRC trailer.

For a payload of N octets, P is `ceil(N / 4)`; the entire frame is `P + 2` flits. CRC-8 covers the header and all transmitted payload flits, including required zero padding.

Preamble, synchronization, idle symbols, and request/grant signaling are physical-medium concerns and are not represented as DLP flits.

DLP has no universal MAC source or destination fields. Point-to-point wiring, a hub port, a GNET-A premise channel, or a GNET-P trunk supplies local delivery identity. Link-local fanout is an explicit control operation, never learned switching.

The maximum payload, CRC polynomial/initial value/reflection, error counters, and foreign-protocol encapsulation details are OPEN. The older possibility of an even smaller discriminator-only DLP header is also retained as an open alternative until this draft is frozen.
