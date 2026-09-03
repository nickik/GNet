---
id: vault-guide
title: "Vault Guide"
aliases: ["How to use the GNet vault"]
type: meta
status: active
tags: ["gnet", "gnet/meta", "gnet/status/active"]
parent: "[[GNet Home]]"
related: ["[[Metadata Schema]]", "[[Tag Index]]", "[[Contributing]]"]
updated: 2026-09-02
---
# Vault guide

> [!info] Knowledge graph
> **Up:** [[GNet Home]] · **Related:** [[Metadata Schema]] · [[Tag Index]] · [[Contributing]]

Open the repository root as an Obsidian vault and begin at [[GNet Home]]. No community plugins are required.

## Information architecture

- **00 Meta** contains status, terminology, contribution rules, and the backlog.
- **01 Architecture** explains system-wide relationships and boundaries.
- **02 Media and Links** defines physical/link families and DLP.
- **03 Protocols** defines protocol semantics and state.
- **04 Packet Formats** defines actual 32-bit wire layouts.
- **05 Registries** assigns numeric types and message identifiers.
- **06 Implementation** separates interoperable behavior from PLIO/QDX mechanisms.
- **07 History** records recovered and superseded project context.
- **08 Decisions** contains architecture decision records.
- **09 Templates** standardizes new notes.

## Reading rules

1. Read [[Specification Status]] before interpreting a field or requirement.
2. Follow the nearest Map of Content rather than browsing folders blindly.
3. Treat a note's YAML `status` as canonical.
4. Use [[Open Questions]] for unresolved work; do not hide uncertainty in protocol prose.
5. Use decision notes when changing a frozen or accepted architectural choice.
6. Packet layouts use RFC-style 32-bit rows. A transmitted flit row shows the 4-bit VCID plus 28 carried bits; logical protocol rows must be labelled `Word`, never `Flit`.

## Linking rules

Use unique note names and Obsidian wikilinks. Link a concept the first time it is materially used. Prefer `[[GDP Protocol]]` over a path-dependent Markdown link. Use aliases only for natural display text.

Each durable note should link upward to one MOC and sideways to two or more closely related notes. Obsidian backlinks then reveal downstream dependencies automatically.
