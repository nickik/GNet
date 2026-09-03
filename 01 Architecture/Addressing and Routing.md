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

Status: **FROZEN principles; OPEN bit allocation and routing wire protocol**

## Address model

A GDP address is an unsigned 64-bit global address. Prefixes aggregate administratively and geographically.

The preferred human-facing hierarchy uses terms such as:

```text
Top / Org / Division / ... / Device
```

The exact intermediate levels and bit partition remain open; `Top`, `Org`, and `Division` replace older `Region`/`Facility` terminology in current design prose. Variable prefix lengths let organizations, campuses, households, and providers receive appropriately sized blocks. Every retail/customer delegation must leave useful local suffix space.

Zero is reserved for an unconfigured/provisional source where a bootstrap profile explicitly permits it. GNet does not define an Ethernet-style global broadcast address.

## Local configuration

1. GLCP establishes the physical/link relationship and capabilities.
2. The endpoint discovers an authorized router using the network bootstrap profile.
3. The router advertises/delegates a prefix and policy information.
4. The endpoint claims/configures an address under that delegation.
5. Subsequent status/configuration reflects changes to the endpoint/router state.

Physical port identity is useful local policy input but is not a globally visible MAC address.

## Routing

Forwarding uses longest/deepest prefix match. A route identifies an egress link/next hop plus policy/metric/validity information.

Routing capability is **not exclusive to a dedicated router product**. Any capable and authorized GNet host may advertise reachability or delegated prefixes. Dedicated routers package forwarding performance, many interfaces, management, and WAN/trunk functions.

Horizontal peering is permitted; a parent/top-level route is fallback, not mandatory transit. The exact route-exchange protocol, authentication, convergence, loop prevention, and delegation encoding remain OPEN.
