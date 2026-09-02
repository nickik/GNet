---
id: gnet-home
title: "GNet Home"
aliases: ["Home", "GNet Knowledge Base"]
type: home
status: active
tags: ["gnet", "gnet/home", "gnet/status/active"]
related: ["[[GNet Architecture Overview]]", "[[Specification Status]]", "[[Open Questions]]"]
updated: 2026-09-02
---
# GNet knowledge base

> [!abstract] Purpose
> This vault is the canonical working knowledge base for the GNet protocol suite. It separates frozen constraints, accepted direction, provisional wire formats, open questions, implementation notes, and recovered project history.

## Start here

- [[GNet Architecture Overview]] — narrative tour through every layer.
- [[GNet Layer Model]] — compact responsibility and exclusion boundaries.
- [[Specification Status]] — what is frozen, accepted, draft, or open.
- [[Open Questions]] — prioritized specification backlog.
- [[Glossary]] — canonical terminology.
- [[Vault Guide]] — how to navigate and extend this knowledge base.

## Maps of content

- [[Architecture MOC]]
- [[Media and Links MOC]]
- [[Protocols MOC]]
- [[Packet Formats MOC]]
- [[Registries MOC]]
- [[Implementation MOC]]
- [[Decisions MOC]]
- [[History MOC]]
- [[Templates MOC]]

## Protocol stack

1. **L1 media:** [[GNET-L]], [[GNET-A]], and [[GNET-P]]
2. **L2 direct framing:** [[Direct Link Protocol]]
3. **L3 routed datagrams:** [[GDP Protocol]] and [[GDP Datagram]]
4. **L4/L5 tunnels and streams:** [[GTS Protocol]] and [[GTS Transport Packets]]
5. **L5 security/session behavior:** [[Security and Identity]], [[Transport and Flows]]
6. **L6 names and identity:** [[GNet Service Model]], [[Discovery and Bootstrap]]
7. **L7 applications:** [[GSC Protocol]], [[GTerm Protocol]]

## Current design backbone

- [[ADR-0001 Layer Boundaries]]
- [[ADR-0002 Minimal GDP Header]]
- [[ADR-0003 Bounded Discovery]]
- [[ADR-0004 Direct Point-to-Point Link]]
- [[ADR-0005 Tunnels and Streams]]
- [[ADR-0006 GNET-L Rate]]
- [[ADR-0007 32-bit Flit Format]]

## Open work

```query
tag:#gnet/status/open
```

Draft work is discoverable with:

```query
tag:#gnet/status/draft
```
