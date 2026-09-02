---
id: metadata-schema
title: "Metadata Schema"
aliases: ["GNet note properties"]
type: meta
status: active
tags: ["gnet", "gnet/meta", "gnet/status/active"]
parent: "[[GNet Home]]"
related: ["[[Vault Guide]]", "[[Tag Index]]", "[[Contributing]]"]
updated: 2026-09-02
---
# Metadata schema

> [!info] Knowledge graph
> **Up:** [[GNet Home]] · **Related:** [[Vault Guide]] · [[Tag Index]] · [[Contributing]]

Every durable Markdown note uses YAML properties.

| Property | Required | Meaning |
|---|---:|---|
| `id` | yes | stable lowercase identifier independent of path |
| `title` | yes | canonical human-readable note title |
| `aliases` | yes | historical names and abbreviations |
| `type` | yes | architecture, protocol, packet, registry, decision, media, implementation, history, meta, backlog, MOC, or home |
| `status` | yes | frozen, accepted, draft, open, mixed, informative, or active |
| `layers` | when applicable | one or more of L1 through L7 |
| `tags` | yes | hierarchical discovery tags |
| `parent` | except Home | wikilink to the owning MOC |
| `related` | recommended | focused lateral wikilinks |
| `updated` | yes | last material revision date |

## Status meanings

- **frozen:** architectural constraint; requires a superseding ADR to change.
- **accepted:** selected direction, possibly awaiting complete wire details.
- **draft:** concrete working proposal that may change.
- **open:** unresolved design.
- **mixed:** the note intentionally contains more than one status.
- **informative:** context, history, or planning rather than a requirement.
- **active:** maintained navigation or process note.
