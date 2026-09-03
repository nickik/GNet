---
id: adr-0015-restore-minimal-gdp-header
title: "ADR-0015 Restore Minimal GDP Header"
aliases: ["Decision 0015","GDP no checksum restored"]
type: decision
status: accepted
layers: ["L3"]
tags: ["gnet","gnet/decision","gnet/status/accepted","gnet/layer/l3"]
parent: "[[Decisions MOC]]"
related: ["[[GDP Protocol]]","[[GDP Datagram]]","[[ADR-0002 Minimal GDP Header]]","[[ADR-0009 No GDP Integrity Field]]"]
updated: 2026-09-03
---
# Decision 0015: Restore the minimal GDP header

Status: **ACCEPTED 2026-09-03**

An intermediate draft added an 8-bit GDP header checksum and a 16-bit Flow Control ID. Both are removed.

GDP semantic fields are limited to:

```text
Version
Type
Size Class
Hop Limit
QoS
64-bit Source
64-bit Destination
```

Reserved wire padding may exist but carries no protocol semantics.

GDP MUST NOT contain checksum/CRC/integrity, link credit state, Flow Control ID, receive window, session identity, fragmentation state, or options.

Rationale:

- DLP already owns hop-local integrity;
- GLCP/DLP own receiver credits and transmission grants;
- GTS/higher layers own transport/session flow state and end-to-end integrity;
- keeping these out of GDP preserves a small router fast path and clean layering.

The existing four-bit GDP Size Class remains because it is package sizing/routing-profile metadata rather than transport flow state.
