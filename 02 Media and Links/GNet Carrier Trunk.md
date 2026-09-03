---
id: gnet-carrier-trunk
title: "GNet Carrier Trunk"
aliases: ["GCT", "GCT25-CX", "GNet trunk"]
type: media
status: draft
layers: ["L1","L2"]
tags: ["gnet","gnet/media","gnet/trunk","gnet/status/draft"]
parent: "[[Media and Links MOC]]"
related: ["[[Direct Link Protocol]]","[[Virtual Channels and VCIDs]]","[[GNet Broadband Access]]"]
updated: 2026-09-03
---
# GNet Carrier Trunk (GCT)

**GCT** is the point-to-point carrier/infrastructure trunk family. It replaces the ambiguous `GNet Link/10` / `GNET-P` commercial naming and is deliberately distinct from the **GNet-10** switched LAN PHY.

A GCT link is a routed or infrastructure adjacency, not a LAN segment.

## Initial family

| Profile | Capacity per direction | Medium | Intended role |
|---|---:|---|---|
| `GCT10-CX` | 10 Mbit/s | dual coax | early/low-cost owned trunk |
| `GCT25-CX` | **25 Mbit/s** | dual hardline coax | standard first-generation carrier trunk |
| `GCT50-CX` | 50 Mbit/s | qualified premium dual coax | later short/high-quality trunk |
| `GCT50-F` | 50 Mbit/s | fiber | later optical trunk |
| `GCT100-F` | 100 Mbit/s | fiber | future/core optical trunk |
| `GCT-56` | 56/64 kbit/s | leased digital carrier | remote/backup |
| `GCT-T1` | 1.544 Mbit/s | DS1/T1 | leased carrier |

The rate is the usable directional GNet link target unless a profile says otherwise.

## GCT25-CX baseline

The preferred first owned carrier trunk is **GCT25-CX**:

```text
25 Mbit/s endpoint A -> endpoint B
25 Mbit/s endpoint B -> endpoint A

Coax A: one direction
Coax B: opposite direction
```

The first physical implementation uses two 75-ohm hardline coaxial cables. `CX` identifies coaxial physical media; exact cable diameter, equalization, repeater spacing, connector and distance qualification remain PHY-validation items rather than universal protocol constants.

## Scaling

Capacity can scale by:

1. deploying additional independent GCT trunks;
2. bonding multiple trunks at a higher layer/profile where standardized;
3. moving to faster qualified GCT profiles;
4. moving the same GNet adjacency to fiber.

Two physically diverse GCT25-CX trunks may provide both aggregate capacity and path redundancy. Because each GCT25-CX already uses two directional coax cables, two fully separate trunks normally imply four coax runs.

## Link semantics

Each direction has independent hop-local VC state. The baseline is the current 32-bit VC2 flit. Downstream capacity is credit controlled: one credit is guaranteed space for one physical flit. Exact carrier framing, clock recovery, keepalive, protection switching and integrity encoding remain profile work.
