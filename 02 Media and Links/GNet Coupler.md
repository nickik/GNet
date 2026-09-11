---
id: gnet-coupler
title: "GNet Coupler"
aliases: ["GC3","GNet shared coupler"]
type: media
status: accepted
layers: ["L1","L2"]
tags: ["gnet","gnet/media","gnet/status/accepted","gnet/coupler"]
parent: "[[Media and Links MOC]]"
related: ["[[GNet Link Control Protocol]]","[[GNet PHY Profiles]]","[[Minimum GNet-3 NIC]]","[[GNet Switch]]","[[GCTL Protocol]]"]
updated: 2026-09-11
---
# GNet Coupler

Status: **ACCEPTED architecture; exact GC3 time quantum and GC3-32 economics OPEN**

A **GNet Coupler (GC)** is the low-cost centrally arbitrated shared-medium GNet LAN device. The first product class is GNet-3:

```text
GC3-8
GC3-16
```

`GC3-32` is a later implementation/economics question, not part of the minimum first product set.

There is deliberately **no general-purpose GC10 LAN profile**. Sites needing more aggregate LAN capacity move to GS3; sites needing faster individual links move to GS10.

## Architectural rule

GC3 is **not a forwarding node and not a credit endpoint**. It maintains no GDP address table, no virtual-circuit table, no receiver-credit state, no packet buffer state and no per-flow scheduler state.

GC3 does not parse GCTL, GDP destinations, GDP Size Class, credits or routing information. All attached receivers observe the shared data stream and decide locally whether a GDP packet is relevant to them.

> **Credits are link-local between adjacent forwarding endpoints. GC3 is transparent to that relationship and does not participate in credits.**

For example:

```text
A --- GC3 --- B

A <--- link-local credits ---> B
```

and for routed traffic:

```text
A --- GC3 --- Router R --- ...

A <--- link-local credits ---> R
```

The remote GDP destination may be globally distant; the credit relationship is only with the adjacent forwarding endpoint.

## Data path

GC3 provides one shared 3 Mbit/s data resource. At most one attached endpoint drives the shared transmit resource at a time. The selected endpoint's data is repeated to all attached receivers.

GC3 requires no packet-store memory, GDP route table, destination lookup, VC allocation, credit table or general-purpose routing CPU.

## WANT and PERMIT control pairs

On GC3 the two dedicated control directions are named by their state semantics rather than as a message protocol:

```text
endpoint -> GC3   WANT
GC3 -> endpoint   PERMIT
```

`WANT=1` continuously means that the endpoint requests ownership of the shared data resource.

`PERMIT=1` continuously means that the endpoint is currently allowed to drive the shared data resource.

These are line states, not framed messages. GC3 does not send `REQUEST`, `GRANT`, `CREDIT`, `END` or other parsed runtime control messages.

A GC3 may provide a fixed bootstrap signature so a NIC can distinguish GC3 from GS3. After GC3 identification, the control pair operates only as WANT/PERMIT state.

## Arbitration and release handshake

GC3 selects one requester using a simple fairness policy, initially round-robin.

Ownership is time-bounded rather than packet-bounded because GDP package sizes differ substantially. The exact baseline time quantum remains OPEN and should be chosen so a typical small packet often fits in one turn while large packets naturally require multiple turns.

When a quantum expires, GC3 deasserts `PERMIT`. The endpoint stops transmission at the defined physical boundary, then MUST deassert `WANT`, even if it still has queued data. Only after that release handshake and after the Coupler has moved on may the endpoint assert `WANT` again for another turn.

The four-phase state sequence is therefore:

```text
WANT   rises
PERMIT rises
PERMIT falls
WANT   falls
```

A large GDP transfer may be paused and resumed across multiple ownership quanta. The Coupler does not need to know where packet boundaries occur.

## Credit and control traffic

Credit control is carried on the normal data path as GCTL traffic, not on the GC3 WANT/PERMIT control pair.

A node needing receive capacity from its adjacent forwarding endpoint sends `GCTL CREDIT_REQUEST`; the adjacent forwarding endpoint responds with `GCTL CREDIT`. Both are ordinary data-path control messages and consume GC3 medium time like other traffic.

GC3 neither captures nor interprets these messages.

## Priority

Minimum GC3 has one arbitration class. There is no GC3 `NORMAL`/`REALTIME` control-pair protocol in the baseline design. More sophisticated QoS is a Switch capability unless a later GC profile explicitly adds a simple physical arbitration extension.

## Package-size policy

GC3 fairness is time-based rather than package-size-based. A large GDP package therefore consumes multiple ownership turns rather than monopolizing the shared medium in one turn.

Any package-size restriction on GC3 should derive from endpoint buffer/PHY limits, not from Coupler scheduler state.
