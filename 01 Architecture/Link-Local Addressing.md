---
id: link-local-addressing
title: "Link-Local Addressing"
type: architecture
status: accepted
layers: ["L3"]
tags: ["gnet", "gnet/addressing", "gnet/bootstrap", "gnet/status/accepted"]
parent: "[[Architecture MOC]]"
related: ["[[Discovery and Bootstrap]]", "[[Addressing and Routing]]", "[[GCTL Protocol]]"]
updated: 2026-09-06
---
# Link-local addressing

Status: **ACCEPTED GNet 0.1 fallback profile**

Every GNet client has a link-local GDP address before it receives any router advertisement. This permits a useful local-only state without factory-programmed endpoint numbers, a router, or a Coupler/Switch address allocator.

## Address format

The reserved non-routable link-local prefix is `FE80::/16` in the 64-bit GDP address space. A client generates a fresh nonzero 48-bit suffix at bootstrap:

```text
|             FE80             |       client-generated suffix       |
|             16 bits          |               48 bits               |
```

The prefix is a protocol constant, not a factory identity. The suffix is neither a MAC address nor a permanent client identity. A client MAY retain it across a local reset, but it MUST NOT assume it remains valid after a new bootstrap.

## Scope and router absence

Link-local GDP addresses MUST NOT be routed, advertised outside their directly attached local domain, or used as a delegated/global prefix. They are valid only on the local GNet attachment domain.

After GLCP reaches active state, a client waits for a GCTL `ADVERTISE(Router)` for the bootstrap interval. If one arrives, ordinary routed prefix delegation proceeds. If no router advertisement arrives before the interval expires, the client enters **link-local-only** state and continues using its generated `FE80::/16` address. Timeout is a local state transition; it sends no additional wire message.

Link-local-only addressing deliberately has no collision-detection protocol in GNet 0.1. Applications that need guaranteed unique local naming require a router-delegated address or a later scoped claim/probe extension.

## Bootstrap messages

The existing GCTL bootstrap exchange remains the wire protocol:

```text
SOLICIT(Router) -> ADVERTISE(Router) -> ADDRESS_OFFER -> ADDRESS_CLAIM -> ADDRESS_ACK/NAK
```

The no-router case introduces no new message encoding. `SOLICIT` and `ADVERTISE` retain their current draft GCTL payload encodings; only the link-local address format and timeout behavior are frozen here.
