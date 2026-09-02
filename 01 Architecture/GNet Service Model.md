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
updated: 2026-09-02
---
# Names, services, and endpoint selection

> [!info] Knowledge graph
> **Up:** [[Architecture MOC]] · **Related:** [[Discovery and Bootstrap]] · [[GTS Protocol]] · [[GSC Protocol]]


Status: **ACCEPTED service model; OPEN wire encoding**

GNet distinguishes four different identifiers:

| Identifier | Scope | Purpose |
|---|---|---|
| GDP address | global, routable | identify the current network endpoint/location |
| Tunnel ID | end-to-end session | identify a transport association independently of a single packet |
| Stream ID | within one tunnel | multiplex independently negotiated data streams |
| Service name or selector | setup and directory | identify the requested logical service |

A service name identifies a logical resource, not necessarily a machine. One name may resolve to several providers. The directory may select or rank providers by reachability, availability, load, location, authorization, or policy.

## Directory service record

The working record model contains:

- service name;
- service type;
- one or more GDP addresses;
- supported terminal or application classes;
- authentication method;
- availability/load or preference;
- location or locality;
- access group or authorization policy;
- validity/lifetime.

The exact binary record, naming grammar, replication, and selection algorithm remain OPEN.

## Ports versus service codes

Earlier transport work used 16-bit source and destination ports. A later candidate replaces the destination port during setup with a short service code—at most seven characters—then uses only Tunnel/Stream identifiers for data. Neither approach is frozen. The eventual design should avoid repeating names in every data packet and should permit a standard service such as system information to publish its callable interface through the directory.

## Location independence

Directory results may refer to local or remote subnets. GTerm and other services must behave uniformly in both cases; discovery is not limited to a LAT-style local broadcast domain.
