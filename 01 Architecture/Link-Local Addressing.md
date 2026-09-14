---
id: link-local-addressing
title: "Link-Local Addressing"
type: architecture
status: accepted
layers: ["L3"]
tags: ["gnet", "gnet/addressing", "gnet/bootstrap", "gnet/status/accepted"]
parent: "[[Architecture MOC]]"
related: ["[[Discovery and Bootstrap]]", "[[Addressing and Routing]]", "[[GCTL Protocol]]", "[[ADR-0019 GCTL Credit and Bootstrap Wire Profile]]"]
updated: 2026-09-14
---
# Link-local addressing

Status: **ACCEPTED GNet 0.1 fallback profile**

Every GNet client has a link-local GDP address before it receives any router advertisement. This permits a useful local-only state without factory-programmed endpoint numbers, a router, or a Coupler/Switch address allocator.

## Address format

The reserved non-routable link-local **wire prefix** is `FE80/16` in the 64-bit GDP address space. A client generates a fresh nonzero 48-bit suffix at bootstrap:

```text
|             FE80             |       client-generated suffix       |
|             16 bits          |               48 bits               |
```

The prefix is a protocol constant, not a factory identity. The suffix is neither a MAC address nor a permanent client identity. A client MAY retain it across a local reset, but it MUST NOT assume it remains valid after a new bootstrap.

The all-zero suffix value is reserved as the link-scoped bootstrap discovery destination:

```text
FE80:0000:0000:0000
```

It is not a normal endpoint address and MUST NOT be routed.

## Canonical text form

GDP addresses render in a compact four-group hexadecimal notation, lowercase hexadecimal, with the longest run of two or more zero groups compressed to `::`. The special wire prefix `FE80/16` always renders as the reserved word `local`; parsers accept it case-insensitively. Thus the first ordinary link-local endpoint value is `local::1`, and a client whose suffix is `0x0123456789AB` renders as `local:123:4567:89ab`.

`local` is a presentation alias only. It does not alter the 64-bit value carried in GDP, which continues to start with `FE80`.

## Scope and router absence

Link-local GDP addresses MUST NOT be routed, advertised outside their directly attached local domain, or used as a delegated/global prefix. They are valid only on the local GNet attachment domain.

After the link reaches active state, a client may send `SOLICIT(Router)` to the reserved bootstrap destination and wait for `ADVERTISE(Router)` for the bootstrap interval. If one arrives, ordinary routed prefix delegation proceeds. If no router advertisement arrives before the interval expires, the client enters **link-local-only** state and continues using its generated `local::/16` address. Timeout is a local state transition; it sends no additional wire message.

Link-local-only addressing deliberately has no collision-detection protocol in GNet 0.1. Applications that need guaranteed unique local naming require a router-delegated address or a later scoped claim/probe extension.

## Bootstrap messages

The GCTL bootstrap exchange is:

```text
SOLICIT(Router) -> ADVERTISE(Router) -> ADDRESS_OFFER -> ADDRESS_CLAIM -> ADDRESS_ACK/NAK
```

The exact GNet 0.1 message layouts and bootstrap destination are frozen by [[ADR-0019 GCTL Credit and Bootstrap Wire Profile]].

The no-router case introduces no additional message encoding.
