---
id: service-type-registry
title: "Service Type Registry"
aliases: ["Discovery services"]
type: registry
status: draft
tags: ["gnet","gnet/registry","gnet/status/draft"]
parent: "[[Registries MOC]]"
related: ["[[Discovery Packets]]","[[GNet Service Model]]","[[Canonical Service Selector]]"]
updated: 2026-09-14
---
# Discovery service registry

> [!info] Knowledge graph
> **Up:** [[Registries MOC]] · **Related:** [[Discovery Packets]] · [[GNet Service Model]] · [[Canonical Service Selector]]

Status: **DRAFT allocations**

This registry classifies roles advertised by the discovery system. It is **not** the GTS Canonical Service Selector namespace and its 16-bit values are not carried in GTS CONNECT.

| Value | Service |
|---:|---|
| 0x0000 | Reserved |
| 0x0001 | Router |
| 0x0002 | Directory |
| 0x0003 | Terminal Server |
| 0x0004 | Boot Server |
| 0x0005 | Time |
| 0x0006 | Identity |
| 0x0007 | Network Management |
| 0x0008 | Session/Reservation Server |
| 0xFFFF | Experimental |

A discovery service type identifies a protocol role, not one vendor product or host name. A discovered endpoint may separately publish a [[Canonical Service Selector]] used by GTS to select the actual service when opening a tunnel.
