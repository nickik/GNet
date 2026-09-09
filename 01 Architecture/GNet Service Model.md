---
id: gnet-service-model
title: "GNet Service Model"
aliases: ["Names and services"]
type: architecture
status: mixed
layers: ["L4","L6","L7"]
tags: ["gnet","gnet/architecture","gnet/status/mixed","gnet/layer/l4","gnet/layer/l6","gnet/layer/l7"]
parent: "[[Architecture MOC]]"
related: ["[[Discovery and Bootstrap]]","[[GTS Protocol]]","[[GSC Protocol]]"]
updated: 2026-09-09
---
# Names, services, and endpoint selection

> [!info] Knowledge graph
> **Up:** [[Architecture MOC]] · **Related:** [[Discovery and Bootstrap]] · [[GTS Protocol]] · [[GSC Protocol]]


Status: **ACCEPTED service-selector model; OPEN directory and exact wire packing**

GNet distinguishes four different identifiers:

| Identifier | Scope | Purpose |
|---|---|---|
| GDP address | global, routable | identify the current network endpoint/location |
| Tunnel ID | end-to-end session | identify a transport association independently of a single packet |
| Stream ID | within one tunnel | multiplex independently negotiated data streams |
| Service selector | setup and directory | identify the requested logical service |

A service name identifies a logical resource, not necessarily a machine. One name may resolve to several providers. The directory may select or rank providers by reachability, availability, load, location, authorization, or policy. Human-readable names are resolved above GTS to binary service selectors; GTS itself does not carry arbitrary service-name strings as its fundamental service identifier.

## Directory service record

The working record model contains:

- service name;
- service type;
- one or more GDP addresses;
- GTS service selector;
- supported terminal or application classes;
- authentication method;
- availability/load or preference;
- location or locality;
- access group or authorization policy;
- validity/lifetime.

The exact binary record, naming grammar, replication, and selection algorithm remain OPEN.

## Variable-width service selectors

GTS does not use fixed 16-bit TCP-style source and destination ports. Service selection is a setup operation, separate from tunnel and stream identity.

A service selector consists logically of:

- a 4-bit Service Size Class;
- a Service ID whose width is selected by that class.

| Size class | Service ID width | Intended character |
|---:|---:|---|
| 0 | 8 bits | very small/common registered services |
| 1 | 16 bits | larger registered or application namespaces |
| 2 | 32 bits | large application/organizational namespaces |
| 3 | 64 bits | sparse or private selectors |
| 4 | 128 bits | very sparse/opaque private selectors |
| 5-15 | reserved | future expansion |

The size class is part of the selector representation; the exact placement of its four bits in a CONNECT or STREAM_OPEN packet remains part of the open GTS wire-format work.

The common class-0 case therefore has only 12 logical selector bits: four bits of size class plus eight bits of Service ID. Implementations may support only the smaller classes when appropriate, while larger systems can use wider selectors without changing the service-selection model.

### Enumeration properties

Small selectors are deliberately enumerable and are suitable for public/common services. Large selectors may be allocated sparsely so that exhaustive service scanning is impractical. A 64-bit or 128-bit selector only gains this property when valid values are sparse or difficult to guess; merely placing sequentially assigned services in a wide field does not prevent enumeration.

This is not cryptographic protection. A passive observer that sees a service selector during setup can learn and later reuse that value. Encryption, authentication, and stronger capability semantics are outside the current service-selector mechanism.

Service enumeration is not required by GTS. A higher-level discovery or directory service may publish selected services, while other selectors can be distributed by configuration, naming systems, or other mechanisms.

## Setup-only use

The Service Selector is carried when a service is selected, in CONNECT or STREAM_OPEN as defined by GTS. Once a tunnel/stream has been established, ordinary DATA packets identify it through tunnel and stream state and do not repeat the Service Selector.

This keeps service naming separate from transport demultiplexing: Tunnel IDs identify transport associations, Stream IDs identify flows within those tunnels, and Service Selectors identify what logical service is requested.

## Location independence

Directory results may refer to local or remote subnets. GTerm and other services must behave uniformly in both cases; discovery is not limited to a LAT-style local broadcast domain.
