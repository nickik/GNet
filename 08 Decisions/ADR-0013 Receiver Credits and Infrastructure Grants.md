---
id: adr-0013-receiver-credits-and-infrastructure-grants
title: "ADR-0013 Receiver Credits and Infrastructure Grants"
aliases: ["Decision 0013","GNet credits"]
type: decision
status: accepted
layers: ["L2"]
tags: ["gnet","gnet/decision","gnet/status/accepted","gnet/flow-control"]
parent: "[[Decisions MOC]]"
related: ["[[GNet Link Control Protocol]]","[[GNet Coupler]]","[[GNet Switch]]","[[Minimum GNet-3 NIC]]"]
updated: 2026-09-03
---
# Decision 0013: Receiver credits and infrastructure grants are distinct

Status: **ACCEPTED 2026-09-03**

## Credit

> **1 GNet credit = guaranteed downstream receive capacity for exactly one physical flit.**

Credits are precise to one flit. Credit-return control messages may batch several returned credits.

## Grant

A GRANT is permission from the local Coupler/Switch/link scheduler to consume some reserved credits now.

```text
GRANT <= min(
    sender remaining demand,
    downstream receiver credits,
    scheduler allowance
)
```

Credit is therefore neither airtime nor a scheduling quantum.

## GC3 request/setup

The minimum Coupler request identifies:

```text
destination
GDP Size Class
priority
```

The GC forwards an RX_REQUEST to the local destination. The receiver advertises actual capacity. The GC reserves that capacity and chooses the actual grant.

The baseline maximum NORMAL GC3 scheduling quantum is eight flits. This is a scheduler value only. REALTIME may be serviced at the next grant boundary without revoking already granted flits.

## Consequence

The same invariant scales from a cheap shared GC3 to hop-by-hop wormhole/cut-through GS fabrics without redefining credit meaning.
