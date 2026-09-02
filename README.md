---
id: github-readme
title: "GNet Repository"
aliases: ["GNet README"]
type: entrypoint
status: active
tags: ["gnet", "gnet/meta", "gnet/status/active"]
related: ["[[GNet Home]]", "[[GNet Architecture Overview]]", "[[Specification Status]]"]
updated: 2026-09-02
---
# GNet

GNet is a clean-slate, hardware-efficient network architecture designed as if Digital Equipment Corporation had begun deploying an integrated local, metropolitan, and wide-area network in the late 1970s and early 1980s.

This repository is both an Obsidian-compatible knowledge base and the working protocol specification.

## Use as an Obsidian vault

1. Clone or download the repository.
2. In Obsidian, choose **Open folder as vault** and select the repository root.
3. Begin with [GNet Home](Home.md).

No community plugins are required. Internal notes use unique filenames, YAML properties, wikilinks, backlinks, nested tags, Maps of Content, and RFC-style 32-bit packet diagrams.

## GitHub navigation

- [Architecture overview](01%20Architecture/GNet%20Architecture%20Overview.md)
- [Layer model](01%20Architecture/GNet%20Layer%20Model.md)
- [Media and link protocols](02%20Media%20and%20Links/Media%20and%20Links%20MOC.md)
- [Protocol index](03%20Protocols/Protocols%20MOC.md)
- [Packet formats](04%20Packet%20Formats/Packet%20Formats%20MOC.md)
- [Registries](05%20Registries/Registries%20MOC.md)
- [Architecture decisions](08%20Decisions/Decisions%20MOC.md)
- [Specification status](00%20Meta/Specification%20Status.md)
- [Open questions](00%20Meta/Open%20Questions.md)

## Design goals

- simple hardware forwarding and fixed-format headers;
- point-to-point or centrally controlled links without dependence on MAC learning or global broadcast;
- one routed datagram format across local, access, and trunk media;
- hierarchical 64-bit global addresses and end-to-end reachability without NAT;
- endpoint-owned sessions, reliability, flow control, fragmentation, and encryption;
- real-time reserved flows, mobility, terminal service, and directory-based discovery;
- protocol-neutral links capable of carrying GDP and legacy/internetwork protocols.

## License

This repository is licensed under the [Mozilla Public License 2.0](LICENSE).
