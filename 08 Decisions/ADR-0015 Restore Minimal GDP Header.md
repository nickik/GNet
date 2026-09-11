---
id: adr-0015-restore-minimal-gdp-header
title: "ADR-0015 Restore Minimal GDP Header"
aliases: ["Decision 0015","GDP no checksum restored"]
type: decision
status: superseded
layers: ["L3"]
tags: ["gnet","gnet/decision","gnet/status/superseded","gnet/layer/l3"]
parent: "[[Decisions MOC]]"
related: ["[[GDP Protocol]]","[[GDP Datagram]]","[[ADR-0002 Minimal GDP Header]]","[[ADR-0009 No GDP Integrity Field]]","[[ADR-0018 GDP Header CRC and Local 16-bit Form]]"]
updated: 2026-09-11
---
# Decision 0015: Restore the minimal GDP header — historical record

Status: **SUPERSEDED by [[ADR-0018 GDP Header CRC and Local 16-bit Form]]**

This ADR records the accepted 2026-09-03 state. An intermediate draft had added an 8-bit GDP header checksum and a 16-bit Flow Control ID; ADR-0015 removed both at that time.

GDP semantic fields were then limited to:

```text
Version
Type
Size Class
Hop Limit
QoS
64-bit Destination
64-bit Source
```

Reserved wire padding could exist but carried no protocol semantics.

The encoding candidate placed Destination before Source, allowing a forwarding node to begin output selection before it had received the source address.

ADR-0015 required GDP to contain no checksum/CRC/integrity, link credit state, Flow Control ID, receive window, session identity, fragmentation state, or options.

Rationale at the time:

- DLP was expected to own hop-local integrity;
- GLCP/DLP owned receiver credits and transmission grants;
- GTS/higher layers owned transport/session flow state and end-to-end integrity;
- keeping these out of GDP preserved a small router fast path and clean layering.

[[ADR-0018 GDP Header CRC and Local 16-bit Form]] supersedes only the relevant current GDP-header outcome: GDP now carries a small header-only CRC-8, Version is 2 bits, Local addresses are 16 bits, and the exact Local/Global fixed-header packing is frozen. GDP still carries no payload-integrity field, Flow Control ID, fragmentation state, receive window, session identity, or options.
