---
id: adr-0016-access-carrier-naming
title: "ADR-0016 Access and Carrier Profile Naming"
type: decision
status: accepted
tags: ["gnet","gnet/decision","gnet/naming"]
parent: "[[Decisions MOC]]"
related: ["[[GNet Broadband Access]]","[[GNet Carrier Trunk]]","[[GNet Mobile Access]]"]
updated: 2026-09-03
---
# ADR-0016 — Access and carrier profile naming

## Decision

Use three distinct technical profile-family names outside the ordinary local-LAN `GNet-N` convention:

- **GBA** — GNet Broadband Access;
- **GCT** — GNet Carrier Trunk;
- **GMA** — GNet Mobile Access.

The first broadband profile is `GBA10`. The preferred first owned point-to-point carrier trunk is `GCT25-CX`.

`GNET-A` and `GNET-P`/`GNet Link/N` are superseded names retained for history. `GNet-10` remains exclusively the switched 10 Mbit/s LAN PHY and must not also denote a carrier trunk.

## Rationale

The old names mixed access, LAN, and point-to-point infrastructure terminology and made `GNet Link/10` ambiguous with GNet-10. The new prefixes identify fundamentally different media/access roles while keeping higher-layer GDP/GNet semantics independent of physical deployment.

## Product boundary

These are protocol/media profile names. Vendor-specific switches, exchanges, radio nodes, cable systems and subscriber devices are outside the normative GNet repository.
