---
id: gdp-protocol
title: "GDP Protocol"
aliases: ["GDP","Global Data Protocol"]
type: protocol
status: frozen
layers: ["L3"]
tags: ["gnet","gnet/protocol","gnet/status/frozen","gnet/layer/l3"]
parent: "[[Protocols MOC]]"
related: ["[[GDP Datagram]]","[[Addressing and Routing]]","[[ADR-0002 Minimal GDP Header]]","[[ADR-0018 GDP Header CRC and Local 16-bit Form]]"]
updated: 2026-09-11
---
# GNet Datagram Protocol (GDP)

Status: **FROZEN initial packet-routing semantics, size classes, address forms, and fixed-header packing**

GDP is the common routed Layer-3 protocol. Its initial profile is deliberately minimal and conventional: every packet is routed from its destination address using longest-prefix match.

## Initial required semantics

- **Version** is a 2-bit GDP wire-version field.
- **Type** is a 4-bit next-protocol discriminator. Initial assignments are `0x1` GCTL and `0x2` GTS; `0x0` and `0x3-0xF` are reserved.
- **Size Class** is a frozen 4-bit selector for one exact GDP payload size from the initial registry.
- **Address Form** is one bit selecting Local or Global representation.
- **Hop Limit** is decremented at each GDP router; Global form uses 8 bits and Local form uses 4 bits.
- **Source** and **Destination** identify the effective endpoints. Global form uses 64 bits each; Local form uses 16-bit identifiers within one known local GDP context.
- **Header CRC-8** detects accidental corruption of immutable GDP header information required by routing and endpoint interpretation.

The previous GDP QoS/Traffic Class field is **not part of the accepted initial semantics**. Its former wire space is reserved until a future extension defines semantics.

GDP MUST NOT acquire a payload checksum, payload CRC, hash, fragmentation state, options, Flow Control ID, sequence number, acknowledgement, receive window, congestion-control state, session identity, or encryption metadata.

DLP/GLCP supply hop-local transmission, VC/link state, credits, and scheduling. GDP supplies only a header-integrity check. Endpoints supply end-to-end payload integrity, reliability, segmentation/reassembly where required, and session state above GDP.

## Address forms

Global GDP carries complete 64-bit Destination and Source addresses. Local GDP carries 16-bit Destination and Source identifiers within one known local GDP prefix/context. Mixed Local/Global source and destination encoding is not supported.

Destination is serialized before Source in both forms.

A router/gateway leaving a Local context constructs a Global-form GDP packet using canonical 64-bit endpoint identities. A router may emit Local form on entry to a Local context only when both endpoints are representable there.

## Header integrity

GDP uses `CRC-8-GNET` as defined in [[GDP Datagram]]:

```text
polynomial  0x07
init        0x00
refin       false
refout      false
xorout      0x00
check       "123456789" -> 0xF4
```

The GDP CRC input is the canonical concatenation, with no padding, of:

```text
Version | Type | Size Class | Address Form | Destination | Source
```

Fields are processed most-significant bit first.

The CRC does not cover Hop Limit, the CRC field itself, or currently Reserved bits. Hop Limit can therefore be decremented by a router without recalculating the CRC.

A failed GDP header CRC invalidates the entire packet. The packet is not routed or passed upward as valid.

GDP CRC-8 is not a payload-integrity mechanism. Corruption confined to the GDP payload is outside GDP's guarantee and may be forwarded. Protocols above GDP provide whatever end-to-end payload integrity they require.

The current DLP definition does not yet provide an independent packet boundary that remains trustworthy when a corrupted GDP header also corrupts Size Class. Exact resynchronization after an invalid GDP header remains an unresolved link/profile rule; receivers must not simply reinterpret following carried data as a fresh GDP header.

## Initial routing

The initial profile requires only directly connected routes, administratively configured static routes, an optional static default route, and longest-prefix matching. No route-exchange protocol, metric system, automatic convergence mechanism, or BGP/IGP equivalent is defined in this version.

Static routing loops are configuration errors. Hop Limit is the mandatory loop-containment mechanism. A router may detect an obvious loop earlier, but such detection is optional.

## Failure handling

Malformed or unforwardable GDP packets are dropped. Where the source is valid and GCTL permits an error response, the defined GCTL reason is returned. This includes header CRC failure, unsupported versions, malformed fields/address forms, no route, unknown destination, Hop Limit expiry, unsupported Size Class on the next link, and forwarding aborts. Error reporting is best effort and never substitutes for transport reliability.

## GTS integrity binding

For GTS payloads, GTS supplies mandatory end-to-end payload integrity independently of the GDP header CRC. Its pseudo-header is bound to the effective GDP endpoint identities and Size Class in both Global and Local representations. A Local representation is expanded to the same effective 64-bit endpoint identity before GTS CRC calculation, so representation changes at routers do not invalidate the end-to-end check.

GDP Type is also included in the GTS pseudo-header. Mutable forwarding fields such as Hop Limit and the Local/Global wire representation are excluded.

The current GTS integrity rule uses CRC-8 only for a future compact GTS encoding capable of using the dedicated 0-byte/3-byte tiny classes, and CRC-32 for the ordinary GTS classes. Those CRCs remain GTS fields/trailers and protect transport content; GDP CRC-8 protects only the GDP header.

The exact packet format is defined in [[GDP Datagram]].
