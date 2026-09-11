---
id: adr-0013-receiver-credits-and-infrastructure-grants
title: "ADR-0013 Link-Local Receiver Credits"
aliases: ["Decision 0013","GNet credits"]
type: decision
status: accepted
layers: ["L2"]
tags: ["gnet","gnet/decision","gnet/status/accepted","gnet/flow-control"]
parent: "[[Decisions MOC]]"
related: ["[[GNet Link Control Protocol]]","[[GNet Coupler]]","[[GNet Switch]]","[[Minimum GNet-3 NIC]]","[[GCTL Protocol]]"]
updated: 2026-09-11
---
# Decision 0013: Receiver credits are strictly link-local

Status: **ACCEPTED 2026-09-11; supersedes the earlier infrastructure-reserved-credit model**

## Credit invariant

> **1 GNet credit = guaranteed receive capacity for exactly one physical flit at the next forwarding endpoint on the current link.**

Credits are **link-local, never end-to-end**. A sender does not wait for credit from the ultimate GDP destination across a multi-hop path. Each forwarding boundary maintains an independent credit relationship with the next forwarding endpoint.

Example:

```text
A --- R1 --- R2 --- R3 --- Z

A  <-> R1   independent credits
R1 <-> R2   independent credits
R2 <-> R3   independent credits
R3 <-> Z    independent credits
```

This permits long-distance traffic to pipeline through the network without a global credit round trip.

Credit-return messages may batch multiple returned credits. Link profiles may use different credit depths to account for link speed, propagation delay and buffering.

## GC3 exception: transparent shared medium

A GC3 Coupler is **not a forwarding endpoint** and therefore does not terminate or maintain credits.

```text
A --- GC3 --- B

A <-------- credits --------> B
```

For routed traffic the adjacent forwarding endpoint is the local router:

```text
A --- GC3 --- R1 --- ... --- Z

A <-------- credits --------> R1
```

The GC3 has no credit table, no reserved-credit state and no knowledge of the GDP destination.

## GS3 forwarding boundary

A GS3 is an active forwarding stage. Its ingress and egress credit relationships are conceptually independent.

```text
A <--- credits ---> GS3 <--- credits ---> B
```

Credit offered by GS3 to A describes GS3's own guaranteed receive capacity on A's link. Credit offered by B to GS3 describes B's receive capacity on the egress link. The switch MUST NOT define A's credit balance as a direct mirror of B's current credit balance.

The switch may use local buffering, cut-through forwarding and output backpressure internally, but each link's credit accounting remains independent.

## Credit request carriage

`GCTL CREDIT_REQUEST` and `GCTL CREDIT` are carried on the normal data path, not on the dedicated physical control pair.

This rule applies to GC3 and GS3 alike.

- On GC3, the Coupler repeats these packets without interpreting them.
- On GS3, the Switch is the adjacent forwarding endpoint and may consume a `CREDIT_REQUEST` addressed through the data path and respond with its own `GCTL CREDIT` for that ingress link.

The physical control pair is reserved for functions that the directly attached infrastructure itself must perform; receiver credit semantics are not encoded there.

## Scheduling is separate

Credit is receive capacity, not permission to use a shared medium or switch path at a particular instant.

GC3 uses the physical `WANT` / `PERMIT` state handshake for medium ownership. GS3 may use separate local path/arbitration mechanisms. Neither changes the meaning of a credit.
