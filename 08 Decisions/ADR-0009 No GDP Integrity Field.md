---
id: adr-0009-no-gdp-integrity-field
title: "ADR-0009 No GDP Integrity Field"
aliases: ["Decision 0009","No GDP checksum"]
type: decision
status: superseded
layers: ["L3"]
tags: ["gnet","gnet/decision","gnet/status/superseded","gnet/layer/l3"]
parent: "[[Decisions MOC]]"
related: ["[[GDP Protocol]]","[[GDP Datagram]]","[[ADR-0015 Restore Minimal GDP Header]]"]
updated: 2026-09-03
---
# Decision 0009: GDP has no integrity field — historical record

Status: **SUPERSEDED as a historical ADR; outcome restored by ADR-0015**

ADR-0009 originally removed checksum/integrity fields from GDP. An intermediate 2026-09-03 draft later introduced an 8-bit GDP header checksum and Flow Control ID, temporarily making this ADR appear superseded.

That intermediate design conflicted with the minimal GDP layer boundary and with the decision to keep receiver credits in GLCP/DLP. [[ADR-0015 Restore Minimal GDP Header]] therefore restores the no-GDP-checksum outcome while recording the intervening design history.

Current integrity split:

- DLP: hop-local accidental-error detection;
- GDP: no checksum/CRC/integrity field;
- GTS or another higher layer: end-to-end integrity/reliability where required.
