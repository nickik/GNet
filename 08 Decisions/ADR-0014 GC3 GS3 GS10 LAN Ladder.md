---
id: adr-0014-gc3-gs3-gs10-lan-ladder
title: "ADR-0014 GC3 GS3 GS10 LAN Ladder"
aliases: ["Decision 0014","GNet LAN product ladder"]
type: decision
status: accepted
layers: ["L1","L2"]
tags: ["gnet","gnet/decision","gnet/status/accepted","gnet/lan"]
parent: "[[Decisions MOC]]"
related: ["[[GNet Coupler]]","[[GNet Switch]]","[[GNet PHY Profiles]]"]
updated: 2026-09-03
---
# Decision 0014: Scale LANs from GC3 to GS3 to GS10

Status: **ACCEPTED 2026-09-03**

The normal GNet LAN infrastructure ladder is:

```text
GC3
cheap shared 3 Mbit/s capacity

    -> more aggregate capacity

GS3
independent 3 Mbit/s switched paths

    -> faster endpoint links

GS10
independent ports negotiating 3 or 10 Mbit/s
```

Initial product sizes are GC3-8/16, GS3-8/16, and GS10-8/16.

There is deliberately **no general-purpose GC10**. A shared faster Coupler would make mixed-speed behavior and aggregate-capacity scaling worse than moving the site to a switch.

Special high-speed Couplers may later exist for constrained cluster/fabric uses, but they do not redefine the normal LAN product ladder.
