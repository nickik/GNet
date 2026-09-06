---
id: gnet-link-control-protocol
title: "GNet Link Control Protocol"
aliases: ["GLCP","GNet link control"]
type: protocol
status: mixed
layers: ["L1","L2"]
tags: ["gnet","gnet/protocol","gnet/status/mixed","gnet/layer/l1","gnet/layer/l2"]
parent: "[[Media and Links MOC]]"
related: ["[[GNet PHY Profiles]]","[[GNet Coupler]]","[[GNet Switch]]","[[Minimum GNet-3 NIC]]","[[GLCP Control Flits]]"]
updated: 2026-09-06
---
# GNet Link Control Protocol (GLCP)

Status: **GNet 0.1 HELLO/CAPABILITIES encoding accepted; remaining operations and electrical line code draft**

GLCP is the hop-local control protocol used by native GNet links. On GNet-3 and GNet-10 copper it runs full-duplex on the dedicated CONTROL-UP and CONTROL-DOWN pairs while data uses DATA-UP and DATA-DOWN.

GLCP is not GDP. It never becomes a routed packet merely to perform local flow control.

## Bootstrap responsibilities

GLCP provides:

- link synchronization and HELLO/presence;
- mandatory Minimum GNet-3 compatibility establishment;
- capability advertisement and selection;
- data-rate and mode selection;
- reset and error recovery;
- link status.

Every advanced NIC begins in the Minimum GNet-3 compatibility mechanism. GNet-10, GNet-20, VC4, larger buffers, and other features are enabled only after both sides agree.

## Runtime operations

The baseline semantic operations are:

| Operation | Purpose |
|---|---|
| `REQUEST` | sender asks to begin/continue a transfer and identifies local destination, GDP Size Class, and priority |
| `RX_REQUEST` | infrastructure asks the destination to reserve receive capacity for the proposed transfer |
| `CREDIT` | receiver advertises guaranteed free capacity in physical flits |
| `GRANT` | infrastructure gives the sender permission to consume some reserved credits now and identifies the VC |
| `END` / `RELEASE` | complete/release transfer and VC state |
| `ABORT` | cancel an active allocation |
| `RESET` | discard link-local control/VC state and restart baseline negotiation; its 0.1 encoding is defined in [[GLCP Control Flits]] |

Minimum GC request semantics are:

```text
REQUEST {
    destination     local link/port identity
    size_class      GDP package Size Class
    priority        NORMAL or REALTIME
}
```

`destination` is the next local attachment, not the final routed GDP destination. For a routed packet leaving the segment, the local destination can therefore be the router while the GDP Destination remains the final endpoint.

## CREDIT versus GRANT

These are deliberately different resources:

> **1 CREDIT = guaranteed downstream receive capacity for exactly one physical flit.**

A receiver may return credit in batches such as `+4`, `+8`, or `+16`; accounting remains exact to one flit.

A GRANT is scheduler permission to transmit now. Infrastructure MUST obey:

```text
GRANT <= min(
    sender remaining demand,
    downstream reserved credits,
    scheduling allowance
)
```

A granted/reserved credit cannot be granted again until the receiver returns it or link recovery cancels the reservation.

## GNet 0.1 bootstrap control flits

[[GLCP Control Flits]] defines the accepted 34-bit logical control-flit layouts for `HELLO` and `CAPABILITIES`, including their generation checks and client-to-infrastructure negotiation sequence. The client begins a newly present link with `HELLO(initial)`; the infrastructure is the selecting authority for its physical port.

## Control timing and remaining encoding work

The current engineering target is approximately **1 Mbit/s logical control signaling per direction** on the dedicated control pairs. A 34-bit logical control flit occupies about 34 microseconds before line-code overhead.

`HELLO` and `CAPABILITIES` have accepted GNet 0.1 opcode/layout definitions. Serialization, exact line code, and the encodings of the remaining operations are **DRAFT — requires PHY validation**. Manchester/biphase-style self-clocking encoding is a historically plausible candidate, not a frozen requirement.

GNet-20 moves these semantics in-band after a negotiated mode transition; its reserved control-symbol/flit encoding remains open.
