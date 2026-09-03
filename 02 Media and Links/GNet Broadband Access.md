---
id: gnet-broadband-access
title: "GNet Broadband Access"
aliases: ["GBA", "GBA10"]
type: media
status: draft
layers: ["L1","L2"]
tags: ["gnet","gnet/media","gnet/access","gnet/status/draft"]
parent: "[[Media and Links MOC]]"
related: ["[[Direct Link Protocol]]","[[Virtual Channels and VCIDs]]","[[GNet Carrier Trunk]]"]
updated: 2026-09-03
---
# GNet Broadband Access (GBA)

**GBA** is the native GNet profile family for centrally scheduled shared broadband access. It is a technical access profile, not the name of any vendor product or cable-system appliance.

## GBA10 baseline

The first profile is **GBA10**.

```text
shared upstream capacity:     10 Mbit/s
shared downstream capacity:   10 Mbit/s
maximum service group:        256 attached premises/endpoints
access method:                central scheduler; no uncontrolled bulk collisions
```

One GBA10 service group is one logical shared bus even when the RF implementation uses multiple physical frequency slots.

### Initial cable/RF plan

The first cable implementation targets approximately two 6 MHz RF slots for upstream data and two 6 MHz RF slots for downstream data, giving roughly 10 Mbit/s useful capacity in each direction across the bonded logical bus. Exact modulation, coding, guard bands and plant qualification remain PHY work.

The scheduler owns upstream transmission opportunities. Real-time/reserved traffic may receive recurring reservations; ordinary packet traffic is sent only when granted.

## Scaling rule

The preferred first scaling mechanism is **service-group splitting**, not indefinitely bonding more spectrum into one larger bus.

```text
GBA10-A   <=256 endpoints
GBA10-B   <=256 endpoints
GBA10-C   <=256 endpoints
```

A future faster GBA profile may be standardized when PHY technology justifies it, but GBA10 remains a bounded compatibility/service unit.

## GNet semantics

GBA carries ordinary GNet/GDP traffic and MUST NOT introduce a second network-layer address architecture. Premise/channel identity, scheduling and ranging are access-local state. The current baseline 32-bit VC2 flit is used unless a future GBA profile explicitly standardizes another negotiated format.

## Open PHY work

- exact RF modulation/coding and FEC;
- ranging and timing adjustment;
- transmit-power control;
- detailed grant/control encoding;
- privacy/security profile interaction;
- plant attenuation/noise/repeater limits;
- exact channel-center/guard-band plan.
