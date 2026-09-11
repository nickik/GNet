---
id: gnet-switch
title: "GNet Switch"
aliases: ["GS3","GS10","GNet switched LAN"]
type: media
status: accepted
layers: ["L1","L2"]
tags: ["gnet","gnet/media","gnet/status/accepted","gnet/switch"]
parent: "[[Media and Links MOC]]"
related: ["[[GNet Link Control Protocol]]","[[GNet PHY Profiles]]","[[GNet Coupler]]","[[Minimum GNet-3 NIC]]","[[GCTL Protocol]]"]
updated: 2026-09-11
---
# GNet Switch

Status: **ACCEPTED architecture; remaining control-pair simplification OPEN**

A **GNet Switch (GS)** provides destination-specific active forwarding. Unlike a Coupler, non-conflicting port pairs can transfer simultaneously.

Initial families:

```text
GS3-8
GS3-16

GS10-8
GS10-16
```

## GS3

Every GS3 attachment operates at up to 3 Mbit/s. A switch with eight ports can therefore sustain several independent 3 Mbit/s conversations at once when their destinations do not conflict.

## GS10

Every GS10 port begins through the Minimum GNet-3 compatibility mechanism. After capability exchange, each port independently upgrades a compatible endpoint to 10 Mbit/s.

```text
Port 1   3 Mbit/s
Port 2  10 Mbit/s
Port 3  10 Mbit/s
Port 4   3 Mbit/s
```

A slow endpoint never reduces unrelated ports.

## Forwarding architecture

After a client announces a usable GDP address, a plain GS records that address against the ingress port. This ephemeral attachment map selects the egress port for local delivery; entries learned from a port are discarded on link-down or reset. A GS does not allocate client addresses.

The preferred switch model is:

- cut-through/wormhole forwarding;
- small flit buffers rather than mandatory whole-packet storage;
- strictly link-local receiver credits;
- independent ingress and egress credit state;
- output/path arbitration;
- simultaneous disjoint paths.

## Link-local credit rule

> **1 credit = guaranteed receive capacity for one physical flit at the next forwarding endpoint on that link.**

Credits are never end-to-end across the routed network.

A GS3 is itself an active forwarding endpoint, so the two sides of a switched path have independent credit relationships:

```text
A <--- credits ---> GS3 <--- credits ---> B
```

Credit that GS3 advertises to A describes **GS3 ingress receive capacity**. Credit that B advertises to GS3 describes **B's receive capacity on the egress link**.

These balances are conceptually independent. GS3 MUST NOT simply ask B for credit and mirror B's available credit back to A. Internal buffers and output backpressure decouple the two links.

This means an ingress may accept flits when the GS has local capacity even if the selected egress is temporarily blocked. Conversely, available downstream credit does not require the GS to advertise more ingress credit when its own ingress/path resources are full.

## Credit request carriage

`GCTL CREDIT_REQUEST` and `GCTL CREDIT` travel on the normal data path, not on the dedicated control pair.

When a client attached to GS3 requests credit for its link to the switch, GS3 captures the applicable `GCTL CREDIT_REQUEST` from the data path and responds with `GCTL CREDIT` representing the switch's own ingress capacity for that link.

The switch does not need to solicit the final local destination's credits before replying to the source. Any egress credit relationship is maintained independently between GS3 and that egress endpoint.

## Routing boundary

A plain GS switches within its local attachment domain. A routed GS variant may combine these ports with a routing processor/function and one or more routed/trunk/WAN uplinks. Routing itself remains a GNet protocol capability and is not defined by the switch silicon.
