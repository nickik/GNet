---
id: gnet-coupler
title: "GNet Coupler"
aliases: ["GC3","GNet shared coupler"]
type: media
status: accepted
layers: ["L1","L2"]
tags: ["gnet","gnet/media","gnet/status/accepted","gnet/coupler"]
parent: "[[Media and Links MOC]]"
related: ["[[GNet Link Control Protocol]]","[[GNet PHY Profiles]]","[[Minimum GNet-3 NIC]]","[[GNet Switch]]"]
updated: 2026-09-03
---
# GNet Coupler

Status: **ACCEPTED architecture; GC3-32 and exact fairness policy OPEN**

A **GNet Coupler (GC)** is the low-cost centrally arbitrated shared-medium GNet LAN device. The first product class is GNet-3:

```text
GC3-8
GC3-16
```

`GC3-32` is a later implementation/economics question, not part of the minimum first product set.

There is deliberately **no general-purpose GC10 LAN profile**. Sites needing more aggregate LAN capacity move to GS3; sites needing faster individual links move to GS10.

## Data path

GC3 provides one shared 3 Mbit/s data resource. It does not need packet-store memory, a GDP route table, or a general-purpose routing CPU.

It maintains only small active-flow state such as:

```text
VCID
source port
destination port
priority
reserved/available receiver credits
sender remaining demand
scheduler state
```

A sender uses GLCP `REQUEST(destination,size_class,priority)`. The GC sends `RX_REQUEST` to the destination. The receiver reports real free capacity with `CREDIT`. The GC then schedules consumption of those credits with `GRANT`.

The receiver decides what is safe; the GC decides what transmits now.

## Scheduling quantum

The baseline maximum NORMAL scheduling quantum is **8 flits**. This is a scheduling value, not a credit unit.

```text
GNet-3: 8 × 32 / 3,000,000 ≈ 85.3 µs
```

The GC MAY grant fewer than eight flits when the packet has fewer remaining or when receiver credit is smaller.

A REALTIME request does not revoke already granted flits. The GC finishes the current quantum, withholds the next NORMAL grant, services eligible REALTIME traffic on another VC, then resumes NORMAL traffic.

The exact anti-starvation budget for sustained REALTIME load remains OPEN.

## Priority

Minimum GC3 has exactly two priorities:

- `NORMAL`
- `REALTIME`

REALTIME is reserved for short latency-sensitive traffic such as voice/control. It is not a generic bulk-traffic priority class.

## Package-size policy

GC3 deliberately limits GDP package sizes. The current baseline profile permits classes **0 through 7** (up to `bulk1K`, 1,024 payload bytes) and rejects classes 8 through 15. REALTIME is legal only for classes **1 through 3** (`tiny3B`, `ctrl32B`, `ctrl64B`).

This makes large/jumbo packages a switched/trunk capability rather than forcing the cheapest Coupler/NIC combination to provision for them.
