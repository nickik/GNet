---
id: adr-0009-no-gdp-integrity-field
title: "ADR-0009 No GDP Integrity Field"
aliases: ["Decision 0009","No GDP checksum"]
type: decision
status: superseded
layers: ["L3"]
tags: ["gnet","gnet/decision","gnet/status/superseded","gnet/layer/l3"]
parent: "[[Decisions MOC]]"
related: ["[[GDP Protocol]]","[[GDP Datagram]]","[[ADR-0015 Restore Minimal GDP Header]]","[[ADR-0018 GDP Header CRC and Local 16-bit Form]]"]
updated: 2026-09-11
---
# Decision 0009: GDP has no integrity field — historical record

Status: **SUPERSEDED historical ADR; its outcome was restored by ADR-0015 and later superseded by ADR-0018**

ADR-0009 originally removed checksum/integrity fields from GDP. An intermediate 2026-09-03 draft later introduced an 8-bit GDP header checksum and Flow Control ID, temporarily making this ADR appear superseded.

That intermediate design conflicted with the then-current minimal GDP layer boundary and with the decision to keep receiver credits outside GDP. [[ADR-0015 Restore Minimal GDP Header]] restored the no-GDP-checksum outcome while recording the intervening design history.

[[ADR-0018 GDP Header CRC and Local 16-bit Form]] later superseded that outcome by adding a header-only GDP CRC-8 while retaining the separation of payload integrity and transport reliability above GDP.

Historical integrity split at the time of ADR-0015:

- DLP: hop-local accidental-error detection was intended;
- GDP: no checksum/CRC/integrity field;
- GTS or another higher layer: end-to-end integrity/reliability where required.

This historical split is not the current normative integrity model; see ADR-0018 and [[GDP Datagram]].
