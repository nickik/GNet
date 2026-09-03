---
id: contributing
title: "Contributing"
aliases: ["Contributing to GNet"]
type: meta
status: active
tags: ["gnet","gnet/meta","gnet/status/active"]
parent: "[[GNet Home]]"
related: ["[[Metadata Schema]]","[[Decision Note Template]]"]
updated: 2026-09-02
---
# Contributing to the GNet specification

> [!info] Knowledge graph
> **Up:** [[GNet Home]] · **Related:** [[Metadata Schema]] · [[Decision Note Template]]


1. State whether a change is architectural, wire-format, algorithmic, or editorial.
2. Never silently change a FROZEN constraint. Add a decision record that explains the incompatibility and migration effect.
3. Mark new numeric values DRAFT until accepted into a registry.
4. For every packet change, update the packet document, relevant registry, and at least one hexadecimal test vector. Actual wire-flit diagrams MUST show the 4-bit VCID plus 28 carried bits; logical 32-bit protocol rows MUST be labelled `Word`, not `Flit`.
5. Use the words MUST, MUST NOT, SHOULD, SHOULD NOT, and MAY only for normative requirements.
6. Keep mechanism at the lowest necessary layer: routers forward GDP; endpoints own sessions and security; directories own names.

Contributions are made under the repository's [Mozilla Public License 2.0](../LICENSE). Whether the standards process needs an additional specification or patent-policy commitment remains an open governance question.
