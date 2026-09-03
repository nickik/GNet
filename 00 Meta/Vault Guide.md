---
id: vault-guide
title: "Vault Guide"
aliases: ["How to use the GNet vault"]
type: meta
status: active
tags: ["gnet","gnet/meta","gnet/status/active"]
parent: "[[GNet Home]]"
related: ["[[Specification Status]]","[[Contributing]]"]
updated: 2026-09-03
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
5. A baseline transmitted flit row is 32 bits with 2-bit VCID + 30 carried bits. There is no SOF bit.
6. A 32-bit protocol-layout aid that is not a physical flit must be labelled `Word`.
7. Historical/chat notes are evidence of design evolution, not current requirements.
