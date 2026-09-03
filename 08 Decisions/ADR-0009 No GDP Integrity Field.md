---
id: adr-0009-no-gdp-integrity-field
title: "ADR-0009 No GDP Integrity Field"
aliases: ["Decision 0009","No GDP checksum"]
type: decision
status: superseded
layers: ["L3"]
tags: ["gnet","gnet/decision","gnet/status/superseded","gnet/layer/l3"]
parent: "[[Decisions MOC]]"
related: ["[[GDP Protocol]]","[[GDP Datagram]]","[[Direct Link Protocol]]"]
updated: 2026-09-03
---
# Decision 0009: GDP has no integrity field

Status: **SUPERSEDED 2026-09-03**

This decision formerly removed all integrity fields from GDP. It is retained for historical context but is no longer normative.

The current GDP header includes an **8-bit lightweight header checksum**. It protects routed GDP header information against accidental corruption but does not protect the GDP payload and is not a security mechanism.

The current integrity split is:

- DLP: link-level CRC-8 for physical/link corruption detection.
- GDP: 8-bit header checksum for the routed header.
- GNet or another GDP-carried protocol: end-to-end payload integrity/reliability as required.

See [[GDP Datagram]] and [[Direct Link Protocol]] for the current format.
