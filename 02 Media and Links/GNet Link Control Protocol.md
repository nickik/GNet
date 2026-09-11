---
id: gnet-link-control-protocol
title: "GNet Link Control Protocol"
aliases: ["GLCP","GNet link control"]
type: protocol
status: mixed
layers: ["L1","L2"]
tags: ["gnet","gnet/protocol","gnet/status/mixed","gnet/layer/l1","gnet/layer/l2"]
parent: "[[Media and Links MOC]]"
related: ["[[GNet PHY Profiles]]","[[GNet Coupler]]","[[GNet Switch]]","[[Minimum GNet-3 NIC]]","[[GCTL Protocol]]"]
updated: 2026-09-11
---
# GNet Link Control Protocol (GLCP)

Status: **UNDER REVISION — bootstrap/capability functions retained; runtime credit/grant model superseded**

GLCP is the directly attached infrastructure-control mechanism used by native GNet links. On GNet-3 and GNet-10 copper, dedicated control pairs exist alongside the data pairs.

The control pair is for functions meaningful to the **immediately attached infrastructure**. It is not the carriage mechanism for node-to-node credit control.

## Architectural separation

The following distinction is now normative:

- **GCTL CREDIT_REQUEST / CREDIT** are carried on the normal data path.
- **Credits are strictly link-local** between adjacent forwarding endpoints.
- **GC3 is not a forwarding endpoint** and does not participate in credit state.
- **GS3 is a forwarding endpoint**; its ingress and egress credit relationships are independent.
- Physical medium/path permission is distinct from receiver credit.

A credit therefore never means permission to use a shared medium or switch path at this instant.

## GC3 control-pair mode

After a NIC identifies that it is attached to GC3, the dedicated control pair ceases to be a parsed message channel and operates as two continuous line states:

```text
endpoint -> GC3   WANT
GC3 -> endpoint   PERMIT
```

`WANT=1` means the endpoint requests ownership of the shared data medium.

`PERMIT=1` means the endpoint may currently drive the shared data medium.

GC3 runtime has no GLCP `REQUEST`, `CREDIT`, `GRANT`, `END`, VC allocation or per-flow control messages.

A fixed bootstrap response/signature MAY be used so a NIC can distinguish a GC3 from a GS3 without requiring a general-purpose parser in the Coupler.

## GS3 control-pair mode

GS3 remains an active switch and therefore may require directly attached control functions that GC3 does not. The exact minimum GS3 control-pair operation set remains under review.

Candidate infrastructure-local responsibilities include:

- presence / link synchronization;
- identifying the attached device as GS rather than GC;
- capability and data-rate negotiation;
- reset / physical error recovery;
- destination/path setup where required for switched forwarding;
- local transmit/path permission.

Receiver credit is explicitly **not** one of these control-pair responsibilities.

## Credit control

The old GLCP `CREDIT` and infrastructure `GRANT` model is superseded.

> **1 GNet credit = guaranteed receive capacity for exactly one physical flit at the next forwarding endpoint on the current link.**

Credit exchange uses GCTL on the data path:

```text
GCTL CREDIT_REQUEST
GCTL CREDIT
```

On GC3 these messages pass transparently between attached forwarding endpoints. On GS3 a credit request for the source-to-switch link is consumed by GS3, and GS3 replies from its own ingress capacity. GS3's downstream credit relationship is independent.

## Bootstrap responsibilities

Before the GC3/GS3 role is known, a minimal common bootstrap may provide:

- link synchronization / presence;
- infrastructure-type identification;
- Minimum GNet-3 compatibility establishment;
- capability advertisement where applicable.

Advanced modes such as GNet-10 may then negotiate additional physical capabilities on GS-class links.

The exact common bootstrap encoding remains open to revision so that GC3 can implement its response as fixed-function logic rather than as a general runtime control protocol.

## Removed runtime operations

The following earlier GLCP runtime operations are no longer baseline link-control operations:

```text
CREDIT
GRANT
END as credit/allocation release
GC3 REQUEST carrying destination/traffic class
GC3 VC allocation
```

Existing numeric layouts for those operations are historical and must not be treated as current GNet 0.1 requirements.

## Design rule

A useful boundary for future revisions is:

> If an operation expresses communication or receive state of a remote/adjacent GNet forwarding endpoint, carry it on the data path. If it exists only so the directly attached Coupler/Switch can operate the physical attachment or switching fabric, it may belong on the control pair.
