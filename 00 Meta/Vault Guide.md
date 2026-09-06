---
id: vault-guide
title: "Vault Guide"
aliases: ["How to use the GNet vault"]
type: meta
status: active
tags: ["gnet","gnet/meta","gnet/status/active"]
parent: "[[GNet Home]]"
related: ["[[Specification Status]]","[[Contributing]]"]
updated: 2026-09-06
---
# Vault guide

Open the repository root as an Obsidian vault and begin at [[GNet Home]].

## Information architecture

- **00 Meta** — status, terminology, contribution rules, backlog.
- **01 Architecture** — system-wide relationships and boundaries.
- **02 Media and Links** — PHY profiles, cabling, connector, GLCP, DLP, Coupler/Switch.
- **03 Protocols** — network/higher protocol semantics.
- **04 Packet Formats** — actual or logical wire layouts.
- **05 Registries** — numeric allocations.
- **06 Implementation** — interoperability/implementation boundary and minimum NIC.
- **07 History** — recovered/superseded context.
- **08 Decisions** — ADRs.

## Reading rules

1. Read [[Specification Status]] before interpreting a field or requirement.
2. Follow the nearest MOC.
3. Treat current accepted ADRs as stronger than old prose.
4. Use [[Open Questions]] for unresolved work.
5. A GNet `Flit` row is exactly 32 data bits. Baseline VC2 is associated link metadata and is not drawn inside that 32-bit row.
6. A `Phit` is a PHY-specific physical transfer unit; do not assume one phit equals one flit.
7. A 32-bit protocol-layout aid that is not itself a DLP flit must be labelled `Word`.
8. Historical/chat notes are evidence of design evolution, not current requirements.
