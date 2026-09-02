---
id: gctl-message-registry
title: "GCTL Message Registry"
aliases: ["Control message registry"]
type: registry
status: draft
tags: ["gnet","gnet/registry","gnet/status/draft"]
parent: "[[Registries MOC]]"
related: ["[[GCTL Protocol]]","[[Discovery Packets]]","[[Address Configuration Packets]]"]
updated: 2026-09-02
---
# GCTL message registry

> [!info] Knowledge graph
> **Up:** [[Registries MOC]] · **Related:** [[GCTL Protocol]] · [[Discovery Packets]] · [[Address Configuration Packets]]


Status: **DRAFT allocations**

| Value | Message |
|---:|---|
| 0x00 | Reserved |
| 0x01 | SOLICIT |
| 0x02 | ADVERTISE |
| 0x10 | ADDRESS_OFFER |
| 0x11 | ADDRESS_CLAIM |
| 0x12 | ADDRESS_ACK |
| 0x13 | ADDRESS_NAK |
| 0x20 | ECHO_REQUEST |
| 0x21 | ECHO_REPLY |
| 0x22 | DESTINATION_UNREACHABLE |
| 0x23 | HOP_LIMIT_EXCEEDED |
| 0xFF | Experimental |

Error payloads and rate limiting are OPEN.
