---
id: gnet-service-model
title: "GNet Service Model"
aliases: ["Names and services"]
type: architecture
status: mixed
layers: ["L4","L6","L7"]
tags: ["gnet","gnet/architecture","gnet/status/mixed","gnet/layer/l4","gnet/layer/l6","gnet/layer/l7"]
parent: "[[Architecture MOC]]"
related: ["[[Discovery and Bootstrap]]","[[GTS Protocol]]","[[Canonical Service Selector]]","[[GSC Protocol]]"]
updated: 2026-09-14
---
# Names, services, and endpoint selection

> [!info] Knowledge graph
> **Up:** [[Architecture MOC]] · **Related:** [[Discovery and Bootstrap]] · [[GTS Protocol]] · [[Canonical Service Selector]] · [[GSC Protocol]]

Status: **ACCEPTED service-selector model; OPEN directory protocol and record packing**

GNet distinguishes four different identifiers:

| Identifier | Scope | Purpose |
|---|---|---|
| GDP address | global, routable | identify the current network endpoint/location |
| Tunnel ID | receiver-local transport state | identify an established GTS tunnel |
| Stream ID | within one tunnel | multiplex reliable data streams |
| Canonical Service Selector (CSS) | tunnel setup and directory | identify the requested logical service |

A service name identifies a logical resource, not necessarily a machine. One name may resolve to several providers. The directory may select or rank providers by reachability, availability, load, location, authorization, or policy.

The directory/naming layer and CSS are separate. A long human-facing service name may resolve to one or more pairs of:

```text
GDP address + CSS
```

GTS uses the CSS only during CONNECT. After the tunnel exists, ordinary transport packets use Tunnel ID and Stream ID and do not repeat the service selector.

## Directory service record

The working record model contains:

- service name;
- service type;
- one or more GDP addresses;
- Canonical Service Selector;
- supported terminal or application classes;
- authentication method;
- availability/load or preference;
- location or locality;
- access group or authorization policy;
- validity/lifetime.

The exact directory binary record, naming grammar, replication, and selection algorithm remain OPEN.

## Canonical Service Selector

CSS is one **128-bit canonical service namespace** with three wire representations:

| Representation | Wire field | Purpose |
|---|---:|---|
| Registered-8 | 8 bits | globally registered common service |
| Short-32 | 32 bits | four-character service mnemonic |
| Full-128 | 128 bits | full canonical selector |

These are compressed representations of the same identity, not separate namespaces.

A Short-32 value is exactly four uppercase ASCII letters/digits and maps into the most-significant 32 bits of CSS128; the low 96 bits are zero.

For example:

```text
#CAPI
    Short-32 = 0x43415049
    CSS128   = 43415049000000000000000000000000
```

Registered-8 values map through the global CSS registry to a four-character Short-32 mnemonic, then to CSS128:

```text
-FILE
    Registered-8 = 0x01
    Short-32     = 0x46494C45   "FILE"
    CSS128       = 46494C45000000000000000000000000

-GRPC
    Registered-8 = 0x02
    Short-32     = 0x47525043   "GRPC"
    CSS128       = 47525043000000000000000000000000
```

The shortest available representation is canonical. Because `FILE` has a Registered-8 assignment, `#FILE` is not a separate service and is not a canonical wire representation.

The complete normative rules are in [[Canonical Service Selector]] and numeric Registered-8 assignments are in [[CSS Registered Service Registry]].

## Endpoint presentation

An endpoint and selected service may be written as:

```text
<GDP-address>:-FILE
<GDP-address>:-GRPC
<GDP-address>:#CAPI
<GDP-address>:#MYEP
<GDP-address>:0123456789ABCDEF0123456789ABCDEF
```

The GDP address answers **where**. CSS answers **which service at that endpoint**.

This notation is presentation syntax; GDP carries only the address and GTS CONNECT carries the CSS.

## Tunnel binding

A GTS CONNECT selects exactly one CSS and simultaneously creates the tunnel and Stream 0.

If accepted, the tunnel is bound to that service identity for its lifetime. Additional STREAM_OPEN exchanges create more streams inside the same selected service. STREAM_OPEN does not select a second service and does not contain CSS.

Selecting another service at the same GDP address requires another GTS tunnel.

This separation keeps service naming out of ordinary data packets:

```text
CONNECT       GDP address + CSS -> create service-bound tunnel
DATA          Tunnel ID + Stream ID
STREAM_OPEN   Tunnel ID + new Stream ID
```

## Enumeration properties

Registered-8 and Short-32 selectors are deliberately enumerable and are suitable for common/public services.

A sparse Full-128 CSS can make blind enumeration impractical when its value is unpredictable, but width is not authentication. A passive observer that learns a CSS may attempt to use it. Authentication and authorization belong to the selected service or a higher security layer.

Service enumeration is not required by GTS. A higher-level discovery or directory service may publish selected services, while private Full-128 selectors may be distributed by configuration or another authorized mechanism.

## Location independence

Directory results may refer to local or remote networks. GTerm and other services must behave uniformly in both cases; discovery is not limited to a local broadcast domain.
