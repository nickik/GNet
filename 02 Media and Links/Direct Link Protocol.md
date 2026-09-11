---
id: direct-link-protocol
title: "Direct Link Protocol"
aliases: ["DLP","GNET-LINK"]
type: protocol
status: mixed
layers: ["L2"]
tags: ["gnet","gnet/protocol","gnet/status/mixed","gnet/layer/l2"]
parent: "[[Media and Links MOC]]"
related: ["[[34-bit Flit Format]]","[[GDP Datagram]]","[[Virtual Channels and VCIDs]]","[[GNet Link Control Protocol]]","[[GCTL Protocol]]"]
updated: 2026-09-11
---
# Direct Link Protocol (DLP)

Status: **ACCEPTED boundary and segment model; invalid-GDP-header resynchronization remains open**

DLP is the minimal Layer-2 data-path contract for one GNet hop. It deliberately avoids global addressing, sessions, routing policy, user identity and application semantics.

DLP also contains **no local addressing field**. Native GNet addressing remains in GDP.

## Baseline flit

Every baseline data flit is:

```text
[ VCID:2 | Carried bits:32 ]
```

There is no SOF field and no first-flit Frame Type field.

The exact VC allocation rules are link-profile dependent and remain under revision for GC3 versus GS3. A GC3 Coupler itself does not allocate or interpret VCs.

## Segment length

Native GNet data segments carry GDP. The GDP Size Class identifies the exact payload budget, allowing receivers and forwarding infrastructure that participate in forwarding to determine the expected bounded transfer without adding a second DLP size-class system.

DLP does **not** define the superseded 64/256/1024-byte Segment Class field. See [[DLP Segment Size Classes]] for historical context and [[GDP Datagram]] for current package-size classes.

Adaptation profiles that carry a non-GDP protocol directly over DLP MUST define an equivalent bounded-length binding.

If the GDP header fails validation, its Size Class cannot be trusted as a packet boundary. The mechanism for establishing the next trustworthy GDP boundary after such a failure remains unresolved; DLP does not infer a new GDP packet start from arbitrary following carried bits.

## Link control separation

Dedicated physical control pairs are infrastructure-local and are defined by [[GNet Link Control Protocol]].

For GC3 they operate as continuous `WANT` / `PERMIT` states after bootstrap. Receiver credit exchange is not carried on the GC3 control pair.

For GS3 the exact remaining control-pair operation set is under review, but receiver credit is likewise not a control-pair operation.

## Flow control

> **1 GNet credit = guaranteed receive capacity for exactly one physical flit at the next forwarding endpoint on the current link.**

Credits are strictly link-local, never end-to-end across the routed network.

A GC3 Coupler is not a forwarding endpoint and therefore does not terminate or maintain credits. Two endpoints communicating across GC3 exchange link-local credits with each other; for routed traffic, the local router is the adjacent forwarding endpoint.

A GS3 is an active forwarding endpoint. Its ingress and egress credit relationships are independent; the source-to-GS credit balance is not a mirror of the downstream destination's balance.

Credit solicitation and return use `GCTL CREDIT_REQUEST` and `GCTL CREDIT` on the normal data path.

Credit is receive capacity, not instantaneous permission to use a medium or switch path. GC3 medium permission is expressed separately by `WANT` / `PERMIT`; GS3 path scheduling is likewise separate from credit semantics.

## Integrity

The baseline DLP data path defines no periodic CRC window, CHECK/count block, or payload-integrity trailer.

GDP provides its own header-only CRC-8 for routing/header interpretation. GDP payload integrity is not guaranteed by DLP or GDP; protocols above GDP provide whatever end-to-end payload integrity they require.

## Link semantics

DLP assumes direct adjacency or a centrally controlled local medium. It does not introduce Ethernet-style MAC source/destination address fields.

On GC3, the shared medium exposes transmitted GDP traffic to all attached receivers; GDP destination semantics determine which nodes or routers consume the packet.
