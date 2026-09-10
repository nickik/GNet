---
id: gdp-protocol
title: "GDP Protocol"
aliases: ["GDP","Global Data Protocol"]
type: protocol
status: mixed
layers: ["L3"]
tags: ["gnet","gnet/protocol","gnet/status/mixed","gnet/layer/l3"]
parent: "[[Protocols MOC]]"
related: ["[[GDP Datagram]]","[[Addressing and Routing]]","[[ADR-0002 Minimal GDP Header]]","[[ADR-0015 Restore Minimal GDP Header]]"]
updated: 2026-09-10
---
# GNet Datagram Protocol (GDP)

Status: **FROZEN initial packet-routing semantics and size classes; local/global wire packing still open**

GDP is the common routed Layer-3 protocol. Its initial profile is deliberately minimal and conventional: every packet is routed from its destination address using longest-prefix match.

## Initial required semantics

- **Version** selects the GDP wire version.
- **Size Class** is a frozen 4-bit selector for one exact GDP payload size from the initial registry.
- **Hop Limit** is decremented at each GDP router; a packet reaching zero is discarded.
- **Source** and **Destination** identify the effective endpoints. The global form is 64 bits each; compact/local representation remains to be frozen.

The previous 8-bit GDP QoS field is **not part of the accepted initial semantics**. Its former wire space remains available/reserved while the first-word encoding is redesigned. A later QoS specification may define explicit semantics and allocate bits without changing the basic routed-datagram model.

The current 4-bit GDP `Type` field is also under review. In existing drafts it is a next-protocol discriminator (`GCTL`, `GTS`, and others); it is **not** the local/global address-mode selector. The initial profile will not freeze that registry until the local/global encoding and minimum required GDP payload demultiplexing are resolved.

GDP MUST NOT acquire a payload-length field, checksum, CRC, hash, integrity flag/trailer, fragmentation state, options, flow/session ID, sequence number, acknowledgement, receive window, congestion-control state, or encryption metadata.

DLP/GLCP supply hop-local transfer framing, integrity, credits, and scheduling. Endpoints supply end-to-end integrity, reliability, segmentation/reassembly where required, and session state above GDP.

## Initial routing

The initial profile requires only directly connected routes, administratively configured static routes, an optional static default route, and longest-prefix matching. No route-exchange protocol, metric system, automatic convergence mechanism, or BGP/IGP equivalent is defined in this version.

Static routing loops are configuration errors. Hop Limit is the mandatory loop-containment mechanism. A router may detect an obvious loop earlier, but such detection is optional.

## Failure handling

Malformed or unforwardable GDP packets are dropped. Where the source is valid and GCTL permits an error response, the defined GCTL reason is returned. This includes unsupported versions, malformed fields/address forms, no route, unknown destination, Hop Limit expiry, unsupported Size Class on the next link, and forwarding aborts. Error reporting is best effort and never substitutes for transport reliability.

## GTS integrity binding

For GTS payloads, GTS supplies mandatory end-to-end CRC protection. Its pseudo-header is bound to the effective GDP endpoint identities and Size Class in both global and compact/local representations. A local representation must be expanded to the same effective endpoint identity before CRC calculation, so representation changes at routers do not invalidate the end-to-end check.

If the GDP `Type` field remains in the final initial header, its value is also included in the GTS pseudo-header. Mutable forwarding fields such as Hop Limit are excluded.

The current GTS integrity rule uses CRC-8 only for a future compact GTS encoding capable of using the dedicated 0-byte/3-byte tiny classes, and CRC-32 for the ordinary GTS classes. The CRC remains a GTS field/trailer and does not change GDP's no-integrity-field rule.

The exact packet format is defined in [[GDP Datagram]].
