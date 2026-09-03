---
id: gnet-infrastructure-naming
title: "GNet Infrastructure Naming"
type: architecture
status: accepted
tags: ["gnet","gnet/naming","gnet/media"]
parent: "[[Media and Links MOC]]"
related: ["[[GNet Broadband Access]]","[[GNet Carrier Trunk]]","[[GNet Mobile Access]]","[[GNet PHY Profiles]]"]
updated: 2026-09-03
---
# GNet infrastructure naming

GNet technical names describe interoperable media/access/link profiles, not vendor product configurations.

| Prefix | Meaning |
|---|---|
| `GNet-3`, `GNet-10`, `GNet-20` | native local switched/coupled PHY generations |
| `GBA` | GNet Broadband Access — centrally scheduled shared broadband access |
| `GCT` | GNet Carrier Trunk — point-to-point carrier/infrastructure link |
| `GMA` | GNet Mobile Access — packet-radio access family |

Examples: `GBA10`, `GCT25-CX`, `GCT-T1`.

Manufacturer product names, vertical-system names, subscriber devices, portable computers and application services are intentionally outside this repository. DEC-specific strategy is maintained at https://github.com/nickik/DEC-Strategy-1975.
