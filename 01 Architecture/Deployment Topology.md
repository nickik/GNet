---
id: deployment-topology
title: "Deployment Topology"
aliases: ["GNet deployment"]
type: architecture
status: draft
layers: ["L1","L2","L3"]
tags: ["gnet","gnet/architecture","gnet/status/draft","gnet/layer/l1","gnet/layer/l2","gnet/layer/l3"]
parent: "[[Architecture MOC]]"
related: ["[[GNET-A]]","[[GNET-P]]","[[Implementation Boundary]]"]
updated: 2026-09-02
---
# Deployment topology and planning profiles

> [!info] Knowledge graph
> **Up:** [[Architecture MOC]] · **Related:** [[GNET-A]] · [[GNET-P]] · [[Implementation Boundary]]


Status: **ACCEPTED topology; DRAFT capacities and product names**

## Layered topology

| Level | Topology |
|---|---|
| Device/home or office | point-to-point GNET-L star through a hub/router |
| Neighbourhood access | shared, centrally scheduled GNET-A coax bus/tree |
| Neighbourhood to district | dedicated point-to-point routed GNET-P trunks |
| District to metro | dual-homed star or partial mesh |
| Metro core | small redundant routed mesh |

The shared-medium model ends at the neighbourhood access plant. District and metro systems are routers, not cable-TV-style shared channels.

## Planning profile

- one neighbourhood loop: 200–300 households, 250 nominal;
- one district: 12 neighbourhood loops or about 3,000 households nominal;
- district expansion: 16 loops or about 4,000 households;
- Boston example: 200 loops, about 50,000 households, and 13–17 district hubs;
- initial neighbourhood/district and district/metro trunks: 10 or 25 Mb/s, with 50 Mb/s as the next GNET-P generation.

These are deployment profiles, not address-bit assignments or protocol limits.

## Draft product vocabulary

Historical planning used **GNET Access 256**, **GNET Trunk 10**, **GNET District 16 (GD16)**, **GNET Core 32 (GC32)**, and **GNET Voice Gateway 96**. The voice gateway was conceived in 96/192/384/768 simultaneous-external-call configurations. These names and capacities are informative until the product strategy freezes them.

## Redundancy and switching

District systems should have at least two independent metro paths; the metro must have at least two core routers. Early systems may scale by adding modest routed nodes and DMA line cards rather than requiring one enormous nonblocking switch. Store-and-forward versus cut-through/wormhole forwarding remains an implementation choice pending timing and buffering analysis.
