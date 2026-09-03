---
id: gctl-protocol
title: "GCTL Protocol"
aliases: ["GCTL","GNet Control Protocol"]
type: protocol
status: draft
layers: ["L3"]
tags: ["gnet","gnet/protocol","gnet/status/draft","gnet/layer/l3"]
parent: "[[Protocols MOC]]"
related: ["[[Discovery Packets]]","[[Address Configuration Packets]]","[[GCTL Message Registry]]","[[GNet Link Control Protocol]]"]
updated: 2026-09-03
---
# GNet Control Protocol (GCTL)

Status: **DRAFT network-control protocol**

GCTL carries network-level discovery, address configuration, routing/OAM, and diagnostic messages. It is distinct from [[GNet Link Control Protocol|GLCP]].

- **GLCP** is hop-local link control: HELLO, capability/rate negotiation, REQUEST, RX_REQUEST, CREDIT, GRANT, VC allocation/release, ABORT, RESET, and link status.
- **GCTL** is network control associated with GDP configuration and routing/service behavior.

The earlier design that used a DLP first-flit Frame Type and direct-DLP GCTL before addressing is superseded. Native GNet bootstrap begins with GLCP. Network discovery/configuration then uses GDP/GCTL with the provisional/link-local addressing rules defined by the bootstrap profile.

GCTL does not provide transport reliability, ordinary directory name resolution, user login, or physical-link credit flow control. Requests that can be retried SHOULD carry a transaction identifier and be idempotent.

Exact bootstrap GDP addressing for an unconfigured endpoint and final GCTL message encodings remain DRAFT.
