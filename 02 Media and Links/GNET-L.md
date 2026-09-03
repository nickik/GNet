---
id: gnet-l
title: "GNET-L"
aliases: ["GNet local star","GNet local copper"]
type: media
status: accepted
layers: ["L1","L2"]
tags: ["gnet","gnet/media","gnet/status/accepted","gnet/layer/l1","gnet/layer/l2"]
parent: "[[Media and Links MOC]]"
related: ["[[GNet PHY Profiles]]","[[GNet Coupler]]","[[GNet Switch]]","[[GNet Link Control Protocol]]"]
updated: 2026-09-03
---
# GNET-L local copper LAN

Status: **ACCEPTED architecture; electrical margins remain under validation**

GNET-L is the native four-pair local GNet attachment used by terminals, workstations, minicomputers, servers, Couplers, and Switches.

The universal profile is [[GNet PHY Profiles|GNet-3]]. Higher-capability endpoints may negotiate GNet-10 on a GS10 port.

```text
Pair 1   CONTROL-UP      endpoint -> GC/GS
Pair 2   CONTROL-DOWN    GC/GS -> endpoint
Pair 3   DATA-UP         endpoint -> GC/GS
Pair 4   DATA-DOWN       GC/GS -> endpoint
```

The connector is [[GNet Modular Connector|GMC-8]]. Cable qualification is defined in [[GNet Copper Cabling]].

GNET-L itself does not define a product topology. A [[GNet Coupler|GC3]] presents one centrally arbitrated shared data resource; a [[GNet Switch|GS3/GS10]] presents independent switched paths.

The former generic "passive/active hub" terminology is superseded by Coupler and Switch because the two devices have materially different scheduling and forwarding behavior.
