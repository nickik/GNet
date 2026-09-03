---
id: gctl-protocol
title: "GCTL Protocol"
aliases: ["GCTL","GNet Control Protocol"]
type: protocol
status: draft
layers: ["L2","L3"]
tags: ["gnet","gnet/protocol","gnet/status/draft","gnet/layer/l2","gnet/layer/l3"]
parent: "[[Protocols MOC]]"
related: ["[[Discovery Packets]]","[[Address Configuration Packets]]","[[GCTL Message Registry]]","[[32-bit Flit Format]]"]
updated: 2026-09-02
---
# GNet Control Protocol (GCTL)

> [!info] Knowledge graph
> **Up:** [[Protocols MOC]] · **Related:** [[Discovery Packets]] · [[Address Configuration Packets]] · [[GCTL Message Registry]] · [[32-bit Flit Format]]


Status: **DRAFT**

GCTL carries bootstrap, discovery, address configuration, and network diagnostic messages. Before address assignment it is carried directly by DLP as link-local GCTL. After assignment it is normally a GDP payload.

In either case, DLP transmits the continuous GCTL/GDP bitstream in 28-bit carried regions, with a 4-bit VCID in every 32-bit flit. Link-local packet notes show their direct-DLP packing; GDP-carried GCTL continues after the GDP header without realignment.

GCTL does not provide ordinary directory name resolution, transport reliability, or user login. A transaction identifier allows a requester to match direct responses to a solicitation. Each request must be safe to repeat because initial delivery is unreliable.

Message families are registered in [[GCTL Message Registry]]. Discovery and address configuration are defined separately in [[Discovery Packets]] and [[Address Configuration Packets]].
