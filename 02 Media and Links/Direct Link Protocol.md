---
id: direct-link-protocol
title: "Direct Link Protocol"
aliases: ["DLP","GNET-LINK"]
type: protocol
status: mixed
layers: ["L2"]
tags: ["gnet","gnet/protocol","gnet/status/mixed","gnet/layer/l2"]
parent: "[[Media and Links MOC]]"
related: ["[[32-bit Flit Format]]","[[GDP Datagram]]","[[Virtual Channels and VCIDs]]","[[GNet Link Control Protocol]]"]
updated: 2026-09-06
---
# Direct Link Protocol (DLP)

Status: **ACCEPTED boundary, flit model, and segment model; DRAFT integrity encoding**

DLP is the minimal Layer-2 data-path contract for one GNet hop. It deliberately avoids global addressing, sessions, routing policy, user identity, and application semantics.

## Baseline flit

Every GNet data flit carries exactly:

```text
[ Data:32 ]
```

Baseline VC2 associates a two-bit hop-local VCID with each flit as link metadata:

```text
VC metadata: [ VCID:2 ]
Flit data:   [ Data:32 ]
```

The VCID is not part of the 32-bit flit data field. There is no SOF field and no first-flit Frame Type field.

The PHY maps each flit plus its associated VC metadata into one or more physical transfer units (phits). An inline serial VC2 implementation therefore conveys 34 logical link bits per flit, but other PHY mappings are permitted where their profile defines the association unambiguously.

The infrastructure allocates a hop-local VC before data is sent. The first data flit received on an inactive allocated VC starts the segment implicitly. The receiver maintains the segment context until the known segment length completes or the VC is aborted, times out, or the link resets.

## Segment length

Native GNet data segments carry GDP. The GDP Size Class identifies the exact payload budget, allowing the receiver and forwarding infrastructure to determine the expected bounded transfer without adding a second DLP size-class system.

For the current 20-octet GDP header, GDP content requires:

```text
header flits  = 5
payload flits = ceil(payload_bytes / 4)
GDP flits     = 5 + ceil(payload_bytes / 4)
```

DLP integrity/trailer flits, if any, are additional and remain dependent on the draft CRC/trailer encoding.

DLP does **not** define the superseded 64/256/1024-byte Segment Class field. See [[DLP Segment Size Classes]] for historical context and [[GDP Datagram]] for current package-size classes.

Adaptation profiles that carry a non-GDP protocol directly over DLP MUST define an equivalent bounded-length binding before using a VC.

## Link control separation

On GNet-3 and GNet-10 copper, [[GNet Link Control Protocol|GLCP]] runs on the dedicated CONTROL-UP and CONTROL-DOWN pairs. GLCP performs bootstrap, capability negotiation, VC allocation, REQUEST/RX_REQUEST, CREDIT, GRANT, release, abort, reset, and link status. These operations are not GDP packets and do not consume data flits.

## Flow control

DLP data transmission obeys actual receiver credits:

> **1 GNet credit = guaranteed downstream receive capacity for exactly one complete 32-bit flit and its associated link metadata.**

Credit is independent of how many phits a PHY uses to transfer that flit. Credit is not permission to transmit immediately. Infrastructure issues a separate GRANT to schedule when some reserved credit may be consumed. See [[GNet Link Control Protocol]], [[GNet Coupler]], and [[GNet Switch]].

## Integrity

Hop-local accidental-error detection belongs to DLP. A small CRC is the intended early implementation. The exact CRC polynomial, initialization/reflection convention, trailer packing, and interaction with a final partially occupied data flit remain **DRAFT — requires interoperability validation**.

GDP has no checksum or integrity field. End-to-end integrity and reliability belong above GDP.

## Link semantics

DLP assumes direct adjacency or a centrally controlled local medium. Local physical identity comes from the actual switch/coupler/router port or medium-supplied channel. DLP therefore has no Ethernet-style MAC source/destination address fields.

VCID state is hop-local and is terminated or reassigned at forwarding nodes.
