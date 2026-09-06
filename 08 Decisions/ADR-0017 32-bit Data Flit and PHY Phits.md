---
id: adr-0017-32-bit-data-flit-and-phy-phits
title: "ADR-0017 32-bit Data Flit and PHY Phits"
aliases: ["Decision 0017","Flit/phit model"]
type: decision
status: accepted
layers: ["L1","L2"]
tags: ["gnet","gnet/decision","gnet/status/accepted","gnet/layer/l1","gnet/layer/l2"]
parent: "[[Decisions MOC]]"
related: ["[[32-bit Flit Format]]","[[Virtual Channels and VCIDs]]","[[GNet PHY Profiles]]","[[ADR-0007 32-bit Flit Format]]","[[ADR-0011 Baseline VC2 Flit Without SOF]]"]
updated: 2026-09-06
---
# Decision 0017: Separate the 32-bit data flit from PHY transfer width and VC metadata

Status: **ACCEPTED 2026-09-06**

## Decision

GNet separates the architectural data/flow-control unit from physical transfer width:

1. A GNet **flit carries exactly 32 data bits**.
2. Baseline **VC2** associates a two-bit hop-local VCID with every flit. Those VCID bits do **not** consume any of the 32 flit data bits.
3. There is **no SOF bit**. The first data flit received on an inactive allocated VC implicitly begins the DLP segment.
4. A **phit** is the physical transfer unit of a particular PHY. One flit may occupy one or more phits, and a phit need not be 32 or 34 bits wide.
5. Each PHY profile defines how the 32-bit flit and its associated VC metadata are serialized, striped, framed, or otherwise conveyed.
6. A simple inline baseline VC2 representation contains `2 VC bits + 32 data bits = 34 logical link bits` per flit. This is not a requirement for a universal 34-bit bus or phit.
7. **1 GNet credit** denotes receive capacity for one complete 32-bit flit and its associated link metadata, independent of the number or width of phits used to convey it.
8. Named GNet data rates such as GNet-3 and GNet-10 denote **32-bit flit-data throughput**. PHY link/symbol rate additionally carries VC metadata and framing/line-code overhead. Inline VC2 alone requires a `34/32` link-bit factor.
9. VC4 or VC8 may be standardized only as **future negotiated options**. Wider VC metadata MUST NOT reduce or otherwise change the 32-bit flit data width.

## Rationale

VCID is purely hop-local forwarding/flow-control state. Making it consume bits from the data flit couples packet packing to a local link resource and causes ordinary 8-, 16-, and 32-bit protocol quantities to straddle flit boundaries.

Keeping a natural 32-bit data flit while associating VC state separately gives NICs, switches, DMA engines, CRC logic, memories, and CPUs a stable 32-bit datapath. It also lets different PHYs choose efficient physical transfer widths without changing DLP/GDP packet packing.

This follows the useful architectural distinction in wormhole/virtual-channel networks between the logical flow-control unit and the physical transfer unit: GNet names these **flit** and **phit** respectively.

## Packet and sizing consequences

The old 30-bit carried-region rule is removed.

For the current GDP format:

```text
GDP header = 20 octets = 160 bits = exactly 5 flits
payload flits = ceil(payload_bytes / 4)
GDP content flits = 5 + ceil(payload_bytes / 4)
```

The GDP payload begins on a flit boundary after the five-flit header. Logical 32-bit words no longer cross data-flit boundaries merely because VC metadata is present.

DLP integrity/trailer encoding remains a separate draft decision and may add trailer flits or define final-flit occupancy rules.

## Compatibility and migration

This change is wire-incompatible with an implementation that interprets the old baseline as a fixed 32-bit transmitted unit containing `VCID:2 + carried:30`. Current implementations and specifications must migrate together to the new flit/phit model.

No endpoint may silently mix the old packed form and the new form. Any future compatibility encapsulation or dual-mode negotiation would require an explicit profile and is not part of this decision.

## Supersession

This ADR:

- supersedes [[ADR-0007 32-bit Flit Format]] insofar as ADR-0007 froze **32 bits as the complete link-transfer width**;
- supersedes [[ADR-0011 Baseline VC2 Flit Without SOF]] insofar as ADR-0011 allocated `2 VCID + 30 carried bits` inside a 32-bit total unit;
- supersedes the historical statement in [[ADR-0008 VCID in Every Flit]] that VCID must be physically inside the fixed-width flit.

It retains the useful earlier decisions that GNet has a 32-bit architectural data unit, that every flit has hop-local VC identity, that baseline VC2 provides four wire VCs, and that segment start is implicit without an SOF bit.
