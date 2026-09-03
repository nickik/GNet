---
id: discovery-packets
title: "Discovery Packets"
aliases: ["SOLICIT and ADVERTISE"]
type: packet
status: draft
layers: ["L3"]
tags: ["gnet","gnet/packet","gnet/status/draft","gnet/layer/l3"]
parent: "[[Packet Formats MOC]]"
related: ["[[GCTL Protocol]]","[[Discovery and Bootstrap]]","[[Service Type Registry]]"]
updated: 2026-09-03
---
# Service discovery packets

Status: **DRAFT — logical message semantics retained; old direct-DLP flit packing superseded**

Discovery is generic: Router, Directory, Terminal Server, and later services use SOLICIT/ADVERTISE with different Service Type values.

These are **GCTL messages carried through the current GDP/GCTL bootstrap profile**. Older diagrams that embedded four-bit VCIDs into each GCTL row are no longer normative because physical-flit packing is now `VC2 + 30 carried bits` and link bootstrap/control belongs to GLCP.

## SOLICIT logical fields

```text
Version
Message Type = SOLICIT
Flags
Transaction ID
Service Type
Scope
```

An unconfigured endpoint first establishes GLCP, then uses the provisional/link-local GDP/GCTL bootstrap rules to solicit an authorized router. Exact bootstrap address encoding remains open.

## ADVERTISE logical fields

```text
Version
Message Type = ADVERTISE
Flags
Transaction ID
Service Type
Preference
Provider Address
Lifetime
Capabilities
```

ADVERTISE echoes the request transaction/service type. A provider address may use bootstrap-specific representation before ordinary address configuration. Lifetime zero means do not cache.

Exact field widths/packing should be re-frozen only after the revised GCTL bootstrap model is settled.
