---
id: gnet-mobile-access
title: "GNet Mobile Access"
aliases: ["GMA"]
type: media
status: draft
layers: ["L1","L2"]
tags: ["gnet","gnet/media","gnet/mobile","gnet/status/draft"]
parent: "[[Media and Links MOC]]"
related: ["[[Direct Link Protocol]]","[[GNet Carrier Trunk]]","[[Addressing and Routing]]"]
updated: 2026-09-03
---
# GNet Mobile Access (GMA)

**GMA** is the GNet access-profile family for centrally managed packet radio. It defines how a radio attachment carries GNet traffic; it does not define a handset, terminal, base-station product, mobile operator, or application service.

## Architectural principles

- Packet data is scheduled rather than treated as a permanently occupied circuit.
- A radio controller may share one data channel among many registered endpoints.
- Radio-local identities, grants, error recovery, power/ranging state and handoff are access-local mechanisms and do not replace GDP addressing.
- Store-and-forward applications may tolerate temporary loss of coverage without changing GMA itself.
- GMA should permit low-duty-cycle endpoints to sleep/listen most of the time and transmit short scheduled bursts.

## Initial narrowband direction

The first implementation may reuse radio technology and channel spacing from contemporary analog cellular systems while assigning selected channels to native packet data. A nominal signaling rate around 10 kbit/s with lower useful payload after FEC, framing and retransmission is a plausible engineering target, but the exact modulation/coding/rate is **not yet frozen**.

## Network boundary

```text
radio access
    |
   GMA
    |
GNet/GDP
    |
routed GNet infrastructure
```

Specific commercial devices and mobile-network system products belong in the external DEC strategy repository, not in this specification.

## Open work

- channel request/grant encoding;
- paging/wakeup mechanism;
- FEC/ARQ profile;
- roaming registration/handoff interaction;
- authentication/privacy;
- exact radio PHY and regulatory profiles;
- mapping between radio scheduling and GNet priority/credit semantics.
