---
id: contributing
title: "Contributing"
aliases: ["Contributing to GNet"]
type: meta
status: active
tags: ["gnet","gnet/meta","gnet/status/active"]
parent: "[[GNet Home]]"
related: ["[[Metadata Schema]]","[[Decisions MOC]]"]
updated: 2026-09-06
---
# Contributing to the GNet specification

1. State whether a change is architectural, wire-format, algorithmic, or editorial.
2. Never silently change an accepted/frozen constraint; add a superseding ADR that records incompatibility and migration effect.
3. Mark unvalidated numeric/electrical values DRAFT until accepted.
4. For packet changes, update the packet document, relevant registry, and eventual golden vectors. A GNet flit diagram MUST show exactly 32 data bits. VCID and other PHY/link metadata MUST be shown separately from the flit data unless documenting a specific PHY encoding. PHY documents MUST distinguish flits from phits.
5. Use MUST/MUST NOT/SHOULD/SHOULD NOT/MAY only for normative requirements.
6. Keep mechanism at the lowest necessary layer: GLCP owns local link control; DLP owns hop transfer/integrity; GDP owns routed package metadata; endpoints own transport/session/security; directories own names.
7. Historical/chat files may preserve obsolete designs but must not be mistaken for normative specification text.

Contributions are made under the repository's [Mozilla Public License 2.0](../LICENSE).
