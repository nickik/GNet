---
id: github-readme
title: "GNet Repository"
aliases: ["GNet README"]
type: entrypoint
status: active
tags: ["gnet","gnet/meta","gnet/status/active"]
related: ["[[GNet Home]]","[[GNet Architecture Overview]]","[[Specification Status]]"]
updated: 2026-09-03
---
# GNet

GNet is a clean-slate, hardware-efficient network architecture designed as if Digital Equipment Corporation had begun deploying an integrated local, metropolitan, and wide-area network in the late 1970s and early 1980s.

This repository is the **authoritative protocol and interoperability specification** for GNet and is also an Obsidian-compatible knowledge base.

## Current baseline

- every native NIC starts as [[Minimum GNet-3 NIC|Minimum GNet-3]];
- baseline flit: **32 bits = 2-bit VCID + 30 carried bits**;
- no SOF bit;
- four copper pairs: control up/down and data up/down;
- receiver-driven flit credits; infrastructure grants are separate;
- `GC3 -> GS3 -> GS10` is the normal LAN scaling ladder;
- GDP remains a minimal routed package with 64-bit addresses and **no GDP checksum or Flow Control ID**.

## Use as an Obsidian vault

Open the repository root as a vault and begin with [GNet Home](Home.md). No community plugins are required. Internal notes use unique filenames, YAML properties, wikilinks, backlinks, Maps of Content, and RFC-style packet diagrams.

## GitHub navigation

- [Architecture overview](01%20Architecture/GNet%20Architecture%20Overview.md)
- [Media and links](02%20Media%20and%20Links/Media%20and%20Links%20MOC.md)
- [Minimum GNet-3 NIC](06%20Implementation/Minimum%20GNet-3%20NIC.md)
- [Protocol index](03%20Protocols/Protocols%20MOC.md)
- [Packet formats](04%20Packet%20Formats/Packet%20Formats%20MOC.md)
- [Architecture decisions](08%20Decisions/Decisions%20MOC.md)
- [Specification status](00%20Meta/Specification%20Status.md)
- [Open questions](00%20Meta/Open%20Questions.md)

## License

This repository is licensed under the [Mozilla Public License 2.0](LICENSE).
