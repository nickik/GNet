---
id: addressing-and-routing
title: "Addressing and Routing"
aliases: ["GNet addressing","GNet routing"]
type: architecture
status: mixed
layers: ["L3"]
tags: ["gnet","gnet/architecture","gnet/status/mixed","gnet/layer/l3"]
parent: "[[Architecture MOC]]"
related: ["[[GDP Protocol]]","[[Address Configuration Packets]]"]
updated: 2026-09-03
---
# Addressing and routing

Status: **FROZEN principles and 0.1 on-link prefix profile; OPEN routing wire protocol**

## Address model

A GDP address is an unsigned 64-bit global address. Prefixes aggregate administratively and geographically.

The preferred human-facing hierarchy uses terms such as:

```text
Top / Org / Division / ... / Device
```

The exact intermediate levels remain open; `Top`, `Org`, and `Division` replace older `Region`/`Facility` terminology in current design prose. Prefixes are carried with an explicit length, rather than imposing one universal `32/32` split.

For GNet 0.1, an address configuration offer MAY use only these routed **on-link** prefix lengths: `/16`, `/32`, `/48`, or `/56`. The router supplies a normalized prefix and confirms one complete 64-bit client address within it. The remaining 48, 32, 16, or 8 bits respectively are the router-managed endpoint space on that link. A `/48` is the ordinary compact leaf-network choice; the other lengths serve larger or smaller directly attached domains.

GNet 0.1 does not define endpoint subnet delegation or a further subnetting protocol. A client receives an address, not authority to subdivide its offered prefix. Other prefix lengths and delegated sub-prefixes remain reserved for a later extension.

Zero is reserved for an unconfigured/provisional source where a bootstrap profile explicitly permits it. GNet does not define an Ethernet-style global broadcast address.

[[Link-Local Addressing]] defines the reserved `FE80::/16` non-routable fallback space. It is separate from globally delegated prefixes and uses a client-generated suffix rather than a factory identity.

## Local configuration

1. GLCP establishes the physical/link relationship and capabilities.
2. The endpoint discovers an authorized router using the network bootstrap profile.
3. The router advertises an on-link prefix from the supported profile and an address candidate within it.
4. The endpoint claims/configures that address under the offered prefix.
5. Subsequent status/configuration reflects changes to the endpoint/router state.

Physical port identity is useful local policy input but is not a globally visible MAC address.

## Routing

Forwarding uses longest/deepest prefix match. A route identifies an egress link/next hop plus policy/metric/validity information.

Routing capability is **not exclusive to a dedicated router product**. Any capable and authorized GNet host may advertise reachability or delegated prefixes. Dedicated routers package forwarding performance, many interfaces, management, and WAN/trunk functions.

Horizontal peering is permitted; a parent/top-level route is fallback, not mandatory transit. The exact route-exchange protocol, authentication, convergence, loop prevention, and delegation encoding remain OPEN.
