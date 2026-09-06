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
updated: 2026-09-06
---
# GNet PHY profiles

## Common flit/phit model

All native GNet PHYs carry the same DLP flit abstraction:

```text
flit data             32 bits
baseline VC metadata   2 bits (VC2)
SOF                     none
```

A **flit** is the 32-bit GNet data/flow-control unit. A **phit** is a PHY-specific physical transfer unit. A PHY may transfer one flit using one or several phits and may encode VC metadata inline, in sideband signaling, or with another profile-defined mechanism.

For a simple inline serial VC2 mapping, one flit requires 34 logical link bits before any additional line coding. This does not make 34 bits a universal physical bus or phit width.

GNet profile data rates are specified as **32-bit flit-data rates**, excluding VC metadata and line-code overhead. Thus an inline VC2 serializer needs `34/32` times the named data rate in link-bit capacity before further coding overhead.

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
flit-data rate       3.0 Mbit/s nominal
fallback rates       1.5 and 0.75 Mbit/s flit data
flit data            32 bits
baseline VCID        2 bits associated metadata
wire VCs             4
control              dedicated up/down pairs
```

If VC2 is serialized inline with no other framing/coding overhead, sustaining these flit-data rates requires approximately:

```text
3.0 Mbit/s flit data    -> 3.1875 Mbit/s link bits
1.5 Mbit/s flit data    -> 1.59375 Mbit/s link bits
0.75 Mbit/s flit data   -> 0.796875 Mbit/s link bits
```

Actual electrical symbol rate may be higher once the final line code/framing is defined.

Every conforming Minimum GNet-3 NIC MUST support 0.75, 1.5, and 3.0 Mbit/s flit-data modes. The fallback ladder is an interoperability choice; it does not imply a guaranteed distance on arbitrary old telephone wiring.

## GNet-10

GNet-10 is a **switched LAN profile**, not a faster shared Coupler.

It retains the four-pair control/data arrangement and baseline VC2 metadata but raises the flit-data rate to 10 Mbit/s. With inline VC2 serialization alone, that corresponds to 10.625 Mbit/s of link bits before additional line coding. A GS10 negotiates each port independently after baseline GNet-3 establishment.

## GNet-20 — FUTURE/DRAFT

GNet-20 is a future bonded copper concept. After explicit capability negotiation, the two control pairs may be repurposed as second data lanes:

```text
DATA-UP lane 0
DATA-UP lane 1
DATA-DOWN lane 0
DATA-DOWN lane 1
```

GLCP semantics would then move in-band via reserved control symbols/phits. The coding, lane deskew, transition sequence, reserved encoding, phit mapping, and recovery behavior are **OPEN** and MUST NOT be inferred from the GNet-3/10 specification.

Future VC4 or VC8 are possible negotiated options only. They do not change the 32-bit flit data width and are not part of GNet-3/10.

## Control-channel engineering target

The dedicated GNet-3/10 control channel currently targets approximately 1 Mbit/s logical signaling per direction. Exact voltage levels, termination, isolation, clock recovery, phit serialization, and line code remain **TBD — requires PHY validation**.
