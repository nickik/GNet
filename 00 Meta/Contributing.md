---
id: contributing
title: "Contributing"
aliases: ["Contributing to GNet"]
type: meta
status: active
tags: ["gnet","gnet/meta","gnet/status/active"]
parent: "[[GNet Home]]"
related: ["[[Metadata Schema]]","[[Decisions MOC]]"]
updated: 2026-09-03
---
# Contributing to the GNet specification

1. State whether a change is architectural, wire-format, algorithmic, or editorial.
2. Never silently change an accepted/frozen constraint; add a superseding ADR that records incompatibility and migration effect.
3. Mark unvalidated numeric/electrical values DRAFT until accepted.
4. For packet changes, update the packet document, relevant registry, and eventual golden vectors. A baseline physical flit diagram MUST show `2-bit VCID + 30 carried bits`; logical 32-bit protocol rows MUST be labelled `Word`, not `Flit`.
5. Use MUST/MUST NOT/SHOULD/SHOULD NOT/MAY only for normative requirements.
6. Keep mechanism at the lowest necessary layer: GLCP owns local link control; DLP owns hop transfer/integrity; GDP owns routed package metadata; endpoints own transport/session/security; directories own names.
7. Historical/chat files may preserve obsolete designs but must not be mistaken for normative specification text.

Contributions are made under the repository's [Mozilla Public License 2.0](../LICENSE).
