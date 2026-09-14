---
id: css-registered-service-registry
title: "CSS Registered Service Registry"
aliases: ["Registered CSS","Registered Service Selectors"]
type: registry
status: frozen
tags: ["gnet","gnet/registry","gnet/status/frozen","gnet/service"]
parent: "[[Registries MOC]]"
related: ["[[Canonical Service Selector]]","[[GTS Protocol]]","[[GNet Service Model]]"]
updated: 2026-09-14
---
# CSS Registered Service Registry

Status: **FROZEN initial allocations; additional allocations may be added compatibly**

Registered-8 CSS values are compact encodings of canonical four-character Short-32 service mnemonics. Every entry therefore defines an 8-bit code, a four-character mnemonic, its 32-bit ASCII value, and the resulting canonical CSS128 value.

| Code | Mnemonic | Short-32 | Canonical CSS128 | Presentation |
|---:|---|---|---|---|
| `0x00` | RESERVED | — | all-zero CSS reserved | — |
| `0x01` | `FILE` | `0x46494C45` | `46494C45000000000000000000000000` | `-FILE` |
| `0x02` | `GRPC` | `0x47525043` | `47525043000000000000000000000000` | `-GRPC` |

All unassigned values are reserved until allocated by a later registry revision.

A registered mnemonic MUST NOT also be used as a canonical Short-32 wire representation. For example, `#FILE` is non-canonical because `-FILE` exists.

The registry is global to the GNet protocol suite. Individual hosts do not reinterpret or locally remap Registered-8 codes.

See [[Canonical Service Selector]] for expansion, canonicalization, wire representation, and presentation rules.
