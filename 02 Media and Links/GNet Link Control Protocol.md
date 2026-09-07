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
| `CREDIT` | receiver advertises guaranteed free capacity in physical flits |
| `GRANT` | infrastructure gives the sender permission to consume some reserved credits now and identifies the VC |
| `ADDRESS_ANNOUNCE` / `ADDRESS_ANNOUNCE_ACK` | client announces a usable GDP address; infrastructure confirms receipt/attachment handling |
| `END` | complete/release transfer and VC state |
| `ABORT` | cancel an active allocation |
| `RESET` | discard link-local control/VC state and restart baseline negotiation; its 0.1 encoding is defined in [[GLCP Control Flits]] |

Minimum GC request semantics are:

```text
REQUEST {
    destination     local attachment / next-hop GDP destination
    traffic_class   NORMAL, REALTIME, CONTROL, or BULK
}
```

`destination` is used by the first Switch/Coupler hop. Once a Switch has selected the destination port, it does not need to forward the original destination field to that client; the downstream request is local to that next hop.

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

[[GLCP Control Flits]] defines the accepted 32-bit logical control-flit layouts for `HELLO` and `CAPABILITIES`, including their generation checks and client-to-infrastructure negotiation sequence. The client begins a newly present link with `HELLO(initial)`; the infrastructure is the selecting authority for its physical port.

## Control timing and remaining encoding work

The current engineering target is approximately **1 Mbit/s logical control signaling per direction** on the dedicated control pairs. A 32-bit logical control flit occupies about 32 microseconds before line-code overhead.

## GNet 0.1 request and credit encoding

The first credit-control profile freezes the `REQUEST` header and `CREDIT` as
32-bit GLCP control flits. A complete `REQUEST` is three consecutive flits:
the header followed by the two 32-bit words of its GDP destination address.
`REQUEST` carries the canonical `traffic-class:8`; package size is learned
from the GDP header once transmission begins and is not duplicated in the
request.

```text
REQUEST header: opcode:4 | version:4 | sender-generation:6 | request-id:6 |
                traffic-class:8 | reserved:4
REQUEST destination: address[63:32] | address[31:0]
CREDIT:  opcode:4 | version:4 | sender-generation:6 | request-id:6 |
          credit-count:8 | reserved:4
GRANT:   opcode:4 | version:4 | sender-generation:6 | request-id:6 |
          vcid:2 | reserved:10
END:     opcode:4 | version:4 | sender-generation:6 | request-id:6 |
          reserved:12
ABORT:   opcode:4 | version:4 | sender-generation:6 | request-id:6 |
          reason:4 | reserved:8
```

Traffic classes are `0x00 NORMAL`, `0x01 REALTIME`, `0x02 CONTROL`, and
`0x03 BULK`. Values `0x04–0xFF` are reserved in 0.1. One credit guarantees
capacity for exactly one physical VC2 flit. A Switch may return a bounded
credit response for an accepted request; credit is capacity, not permission to
transmit. `GRANT` authorizes the sender to consume its currently outstanding
credits on `vcid`; it carries no duplicate flit count. The sender must not
transmit beyond its current credit balance. `END` completes the package after
all required grants and data have been sent; a `GRANT` is not an implicit end
marker.

`HELLO`, `CAPABILITIES`, `REQUEST`, `CREDIT`, `GRANT`, `END`, and `ABORT` have accepted GNet 0.1 opcode/layout definitions. `ABORT` reasons are `0x0 SENDER_ABORT`, `0x1 TIMEOUT`, `0x2 PROTOCOL_ERROR`, and `0x3 RESOURCE_ERROR`; `0x4–0xF` are reserved. Serialization, exact line code, and the encodings of the remaining operations are **DRAFT — requires PHY validation**. Manchester/biphase-style self-clocking encoding is a historically plausible candidate, not a frozen requirement.

GNet-20 moves these semantics in-band after a negotiated mode transition; its reserved control-symbol/flit encoding remains open.
