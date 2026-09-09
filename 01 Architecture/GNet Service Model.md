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

A service name identifies a logical resource, not necessarily a machine. One name may resolve to several providers. The directory may select or rank providers by reachability, availability, load, location, authorization, or policy. GTS itself supports compact binary or fixed-width textual Service Selectors; higher-level directories may map longer human-readable names to them.

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

A Service Selector begins with a 2-bit Size Class. The class determines the representation that follows:

| Size class | Selector field | Representation | Intended use |
|---:|---:|---|---|
| 0 | 8 bits | numeric service code | common registered services and very small systems |
| 1 | 32 bits | 4 ASCII characters | compact named services |
| 2 | 128 bits | 16 ASCII characters | sparse, private, or opaque named services |
| 3 | reserved | — | future expansion |

ASCII names occupy fixed-width byte fields. Names shorter than the field are terminated and padded with zero bytes. The exact allowed character set and case rules remain OPEN.

The common class-0 case therefore requires only 10 logical selector bits: a 2-bit class plus an 8-bit Service ID. A small implementation may support only class 0, while larger systems can support named service selectors without changing the GTS service-selection model.

Examples:

```text
00 + 05                        -> registered service 5, e.g. FILE
01 + "FILE"                    -> four-character named service
10 + "oooooofilesy\0\0\0\0" -> private 128-bit selector field
```

The textual forms are still exact selector values, not names that GTS must further resolve.

### Enumeration properties

The 8-bit namespace is deliberately enumerable and is suitable for public/common services. The 32-bit and especially 128-bit textual namespaces can be allocated sparsely so that exhaustive scanning is impractical.

The width does not by itself make a selector secret. Predictable names can be guessed with a dictionary regardless of the size of the field. A private selector intended to resist scanning should therefore be sufficiently unpredictable within its namespace.

This is not cryptographic protection. A passive observer that sees a Service Selector during setup can learn and later reuse that value. Encryption and authentication are outside the current baseline.

Service enumeration is not required by GTS. A higher-level discovery or directory service may publish selected services, while private selectors can be distributed by configuration or other mechanisms.

## Setup-only use

The Service Selector is carried when a service is selected, in CONNECT or STREAM_OPEN as defined by GTS. Once a tunnel/stream has been established, ordinary DATA packets identify it through tunnel and stream state and do not repeat the Service Selector.

This keeps service naming separate from transport demultiplexing: Tunnel IDs identify transport associations, Stream IDs identify flows within those tunnels, and Service Selectors identify what logical service is requested.

## Location independence

Directory results may refer to local or remote subnets. GTerm and other services must behave uniformly in both cases; discovery is not limited to a LAT-style local broadcast domain.
