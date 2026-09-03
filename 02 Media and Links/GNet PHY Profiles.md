---
id: gnet-phy-profiles
title: "GNet PHY Profiles"
aliases: ["GNet-3","GNet-10","GNet-20","GNet physical profiles"]
type: media
status: mixed
layers: ["L1"]
tags: ["gnet","gnet/media","gnet/status/mixed","gnet/layer/l1"]
parent: "[[Media and Links MOC]]"
related: ["[[GNet Copper Cabling]]","[[GNet Modular Connector]]","[[GNet Link Control Protocol]]","[[Minimum GNet-3 NIC]]"]
updated: 2026-09-03
---
# GNet PHY profiles

## Common baseline copper pairs

Normal GNet-3 and GNet-10 copper use four balanced pairs:

```text
Pair 1   CONTROL-UP      NIC -> GC/GS
Pair 2   CONTROL-DOWN    GC/GS -> NIC
Pair 3   DATA-UP         NIC -> GC/GS
Pair 4   DATA-DOWN       GC/GS -> NIC
```

Control is full duplex and remains available while data is flowing.

## GNet-3

GNet-3 is the universal native-copper compatibility baseline:

```text
nominal data rate    3.0 Mbit/s
fallback rates       1.5 and 0.75 Mbit/s
physical flit        32 bits
baseline VCID        2 bits
carried bits         30 bits
control              dedicated up/down pairs
```

Every conforming Minimum GNet-3 NIC MUST support 0.75, 1.5, and 3.0 Mbit/s data modes. The fallback ladder is an interoperability choice; it does not imply a guaranteed distance on arbitrary old telephone wiring.

## GNet-10

GNet-10 is a **switched LAN profile**, not a faster shared Coupler.

It retains the four-pair control/data arrangement and VC2 flit format but raises the data rate to 10 Mbit/s. A GS10 negotiates each port independently after baseline GNet-3 establishment.

## GNet-20 — FUTURE/DRAFT

GNet-20 is a future bonded copper concept. After explicit capability negotiation, the two control pairs may be repurposed as second data lanes:

```text
DATA-UP lane 0
DATA-UP lane 1
DATA-DOWN lane 0
DATA-DOWN lane 1
```

GLCP semantics would then move in-band via reserved control symbols/flits. The coding, lane deskew, transition sequence, reserved encoding, and recovery behavior are **OPEN** and MUST NOT be inferred from the GNet-3/10 specification.

## Control-channel engineering target

The dedicated GNet-3/10 control channel currently targets approximately 1 Mbit/s logical signaling per direction. Exact voltage levels, termination, isolation, clock recovery, and line code remain **TBD — requires PHY validation**.
