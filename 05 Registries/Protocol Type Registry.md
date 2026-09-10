---
id: protocol-type-registry
title: "Protocol Type Registry"
aliases: ["DLP and GDP protocol types"]
type: registry
status: mixed
tags: ["gnet","gnet/registry","gnet/status/mixed"]
parent: "[[Registries MOC]]"
related: ["[[Direct Link Protocol]]","[[GDP Protocol]]","[[GCTL Protocol]]"]
updated: 2026-09-10
---
# Protocol type registry

Status: **FROZEN initial GDP allocations; DLP adaptation values remain OPEN**

## Native DLP carriage

Native GNet data VCs normally carry GDP. GLCP owns native hop-local link control and GCTL is GDP-carried network control.

Non-GDP DLP adaptation profiles MAY define an explicit protocol selector in their VC/allocation metadata. Exact numeric DLP adaptation values remain OPEN and are not frozen by this table.

## GDP Type registry — initial traditional profile

GDP Type is a 4-bit next-protocol discriminator. The initial packet-routed profile defines only GCTL and GTS.

| Value | Name | Meaning |
|---:|---|---|
| `0x0` | RESERVED | invalid/unassigned |
| `0x1` | GCTL | routed/bootstrap network control, errors, and diagnostics |
| `0x2` | GTS | GNet transport/session |
| `0x3`-`0xF` | RESERVED | undefined in the initial profile |

A receiver MUST drop a GDP packet whose Type is reserved or undefined. When a valid routable source is available and GCTL error-generation rules permit a reply, it reports `DESTINATION_UNREACHABLE` code 2 (unsupported GDP payload type).

Higher-level facilities such as terminal service, directory/name lookup, RPC, voice signaling, and boot protocols are carried through GTS or GCTL as appropriate rather than consuming additional GDP Type values in the initial profile.
