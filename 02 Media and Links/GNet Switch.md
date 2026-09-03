---
id: gnet-switch
title: "GNet Switch"
aliases: ["GS3","GS10","GNet switched LAN"]
type: media
status: accepted
layers: ["L1","L2"]
tags: ["gnet","gnet/media","gnet/status/accepted","gnet/switch"]
parent: "[[Media and Links MOC]]"
related: ["[[GNet Link Control Protocol]]","[[GNet PHY Profiles]]","[[GNet Coupler]]","[[Minimum GNet-3 NIC]]"]
updated: 2026-09-03
---
# GNet Switch

Status: **ACCEPTED architecture**

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

Every GS10 port begins through the Minimum GNet-3 compatibility mechanism. After GLCP capability exchange, each port independently upgrades a compatible endpoint to 10 Mbit/s.

```text
Port 1   3 Mbit/s
Port 2  10 Mbit/s
Port 3  10 Mbit/s
Port 4   3 Mbit/s
```

A slow endpoint never reduces unrelated ports.

## Forwarding architecture

The preferred switch model is:

- cut-through/wormhole forwarding;
- small flit buffers rather than mandatory whole-packet storage;
- hop-by-hop receiver credits;
- per-VC state;
- output/path arbitration;
- simultaneous disjoint paths.

The same invariant applies at every hop:

> **1 credit = guaranteed downstream capacity for one physical flit.**

An input may advance only when the next stage/output has credit and the switch scheduler grants the path. Credit exhaustion creates backpressure instead of packet loss.

## Routing boundary

A plain GS switches within its local attachment domain. A routed GS variant may combine these ports with a routing processor/function and one or more routed/trunk/WAN uplinks. Routing itself remains a GNet protocol capability and is not defined by the switch silicon.
